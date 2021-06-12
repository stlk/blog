---
layout: post
title: Websockets on Django and React
---

For our new Shopify app I needed to create a websocket server that broadcasts a message when model is updated. I ended up with this solution. Start with installing `channels` and `channels-redis` to you Django app.

Add `channels` to `INSTALLED_APPS` and configure `ASGI_APPLICATION` and `CHANNEL_LAYERS`.

myproject/settings.py
```python
INSTALLED_APPS = [
    ...
    "channels",
    ...
]

WSGI_APPLICATION = "myproject.wsgi.application"
ASGI_APPLICATION = "myproject.asgi.application"

CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [os.environ.get("REDIS_URL")],
        },
    },
}
```

[Setup the ASGI](https://channels.readthedocs.io/en/latest/deploying.html#configuring-the-asgi-application) application so it's production ready. Based on [How to deploy with ASGI](https://docs.djangoproject.com/en/3.1/howto/deployment/asgi/) I replaced waitress with daphne.

collection_sorting/asgi.py
```python
import os

from django.core.asgi import get_asgi_application

# Fetch Django ASGI application early to ensure AppRegistry is populated
# before importing consumers and AuthMiddlewareStack that may import ORM
# models.
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")
django_asgi_app = get_asgi_application()

from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter

import myapp.routing

application = ProtocolTypeRouter(
    {
        "http": django_asgi_app,
        "websocket": AuthMiddlewareStack(URLRouter(myapp.routing.websocket_urlpatterns)),
    }
)
```

Create `routing.py` to hook up the consumer.

myapp/routing.py
```python
from django.urls import re_path

from . import consumers

websocket_urlpatterns = [
    # We use re_path() due to limitations in URLRouter.
    re_path(r"ws/job-status/$", consumers.JobStatusConsumer.as_asgi()),
]
```

What is a consumer? This is a description form Django Channels documentation.

> When Django accepts an HTTP request, it consults the root URLconf to lookup a view function, and then calls the view function to handle the request. Similarly, when Channels accepts a WebSocket connection, it consults the root routing configuration to lookup a consumer, and then calls various functions on the consumer to handle events from the connection.

This consumer accepts connections when user is signed in and adds them to their own group. When it receives `propagate_status` message it forwards it to all subscribers.

myapp/consumers.py
```python
import json

from channels.generic.websocket import AsyncWebsocketConsumer


class JobStatusConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        if self.scope["user"].is_anonymous:
            await self.close()
            return

        self.user = self.scope["user"]
        self.group_name = f"job-posting-{self.user.id}"

        await self.channel_layer.group_add(self.group_name, self.channel_name)
        await self.accept()

    async def propagate_status(self, event):
        if not self.scope["user"].is_anonymous:
            message = event["message"]
            await self.send(text_data=json.dumps(message))
```

Now in model's save method we call `group_send` to publish the update.

myapp/models.py
```python
import channels.layers
from asgiref.sync import async_to_sync

class JobPosting(models.Model):

    ...

    def save(self, *args, **kwargs):
    super().save(*args, **kwargs)
    channel_layer = channels.layers.get_channel_layer()
    group = f"job-posting-{self.user.id}"
    async_to_sync(channel_layer.group_send)(
        group,
        {
            "type": "propagate_status",
            "message": {"id": self.id, "state": self.state},
        },
    )
```

Now when everything is wired up I created very simple client to make sure messages are being received.


```js
const updatesSocket = new WebSocket(`ws://${window.location.host}/ws/job-status/`);

updatesSocket.onmessage = function(e) {
    const data = JSON.parse(e.data);
    console.log(data);
};

updatesSocket.onclose = function(e) {
    console.error('Chat socket closed unexpectedly');
};
```

When the server side was working I used `react-use-websocket` to add live updates to React.

```js
import React, { useState } from 'react';
import useWebSocket from 'react-use-websocket';

export default function List() {
  const [data, setData] = useState([]);

  useWebSocket(`wss://${window.location.host}/ws/job-status/`, {
    onMessage: (e) => {
      const message = JSON.parse(e.data);
      setData((data) => (data.map((item) => {
          if (item.id === message.id) {
            item.state = message.state;
          }
          return item;
        }));
    },
    shouldReconnect: (closeEvent) => true,
  });

  return (<div></div>);
}

```