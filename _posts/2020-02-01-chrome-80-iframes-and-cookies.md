---
layout: post
title: Chrome 80, iframes and cookies
---

Google is always kind enough to provide us with something to do. This time it's about Chrome defaulting cookies to [SameSite=Lax](https://web.dev/samesite-cookies-explained/) and a relatively new `SameSite=None` option.

I'm building Shopify apps using Django. The app is embedded inside stores admin meaning [I can't use Strict or Lax](https://help.shopify.com/en/api/guides/samesite-cookies). Django did the same thing and on [August 2018](https://docs.djangoproject.com/en/dev/releases/2.1/#django-contrib-sessions) added `SESSION_COOKIE_SAMESITE` which defaults to Lax. The way I see it changing defaults on the server-side is a good step. However, this change on the browser side came just too quickly for the ecosystem to adapt. The frameworks haven't [caught up yet](https://docs.djangoproject.com/en/dev/ref/settings/#std:setting-SESSION_COOKIE_SAMESITE) and even if they did they would have to consider [incompatible browsers](https://www.chromium.org/updates/same-site/incompatible-clients).

I and [tystar](https://github.com/tystar86) wrote a [middleware](https://github.com/stlk/django-toolbox/blob/master/django_toolbox/cookies_middleware.py) for Django 2.2.x and 3.x to handle this. This middleware should be placed before any other middleware.

```python
MIDDLEWARE = [
    "django_toolbox.cookies_middleware.SamesiteCookieMiddleware",
```