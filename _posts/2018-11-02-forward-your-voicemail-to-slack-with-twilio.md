---
layout: post
title: Forward your voicemail to Slack with Twilio
---

This post expands on the previous one I wrote on this topic. Please refer back to [Virtual support phone number with Twilio]({{ site.baseurl }}{% post_url 2018-09-09-virtual-business-phone-number-with-twilio %}) if you're not familiar with Twilio.

> If you don't want to bother with setup you can use a service I created to do the job for you.
>
> [Visit Support line](https://supportline.rousek.name/)

Following up from last time. We have voice-enabled phone number *+420999999999*. Now We'll setup call forwarding and voicemail for this number.

### Twilio Functions

Functions are a way to run your logic directly on Twilio servers. We'll use them to construct TwiML response and then to send a message to Slack.

### Recording voicemail message

[Runtime > Functions](https://www.twilio.com/console/runtime/functions/manage) is the place where we can manage our Functions.

Create a new function called `Record message` and paste following code. Replace `YOUR_RUNTIME_DOMAIN` with your Runtime Domain from [Runtime Overview](https://www.twilio.com/console/runtime/overview) page.

```javascript

exports.handler = function(context, event, callback) {
  let twiml = new Twilio.twiml.VoiceResponse();

  // Use <Dial> to forward the call to my phone
  const dial = twiml.dial({
    callerId: '+420999999999',
  });
  dial.number('+420111111111');

  // Use <Say> to give the caller some instructions
  twiml.say('We\'re not available right now. Please leave a message after the beep.');

  // Use <Record> to record the caller's message
  twiml.record({action: 'https://YOUR_RUNTIME_DOMAIN/slack'});

  // End the call with <Hangup>
  twiml.hangup();

  // return the TwiML
  callback(null, twiml);
};
```

<p class="post__image-center">
  <img src="/public/twilio-function.png" alt="" class="post__image post__image-full-width">
</p>


You probably noticed that `record` command is pointing to `/slack` URL. This will be another function. But before we can create it we need to add new npm package to our [Runtime > Functions > Configure](https://www.twilio.com/console/runtime/functions/configure). The package is called [axios](https://github.com/axios/axios) and will help us with making HTTP request to Slack.

<p class="post__image-center">
  <img src="/public/twilio-function-config.png" alt="" class="post__image post__image-full-width">
</p>

There's one more thing before we can get to the function and that is Slack Webhook URL. Luckily you can get one by visiting [Slack App Directory](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks). After that, we'll have everything we need to create a Function.

```javascript
var axios = require("axios");

exports.handler = function(context, event, callback) {
  axios
    .post(
      "https://hooks.slack.com/services/TOKEN",
      {
        text: `You have received a message from ${
          event.From
        }.\nTo listen to this message, please visit this URL:${
          event.RecordingUrl
        }.mp3`
      }
    )
    .then(function(response) {
      console.log(response);
      callback(null, "OK");
    })
    .catch(function(error) {
      console.log(error);
      callback(null, "FAIL");
    });
};
```

This Function takes the phone number of the caller and recording URL. Then POSTs a message to Slack. Now you'll get a message when somebody leaves you a message.