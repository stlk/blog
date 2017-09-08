---
layout: post
title: Going headless with Firefox - since '55
---

Recently I found that Firefox supports [headless mode](https://developer.mozilla.org/en-US/Firefox/Headless_mode) on [Linux](https://developer.mozilla.org/en-US/Firefox/Releases/55) for a month now. [Windows and MacOS](https://developer.mozilla.org/en-US/Firefox/Releases/56) are now in beta and will be released on September 26, 2017. I was pretty excited to find out so I immediately switched over Travis configuration for Selenium tests in my Django project.

When running CI in windowed I had to setup virtual framebuffer in `before_install` of `.travis.yml`.

```
export DISPLAY=:99.0
sh -e /etc/init.d/xvfb start
sleep 3
```

Meaning that by going headless I could save at least three seconds of CI runtime!

### How to do it?

All we need to do is to make sure that we are using at least Firefox 55 and then set `MOZ_HEADLESS=1` environment variable. That's right. One environment variable!

My entire `.travis.yml` from [auto-fisera](https://github.com/stlk/auto-fisera) project.

{% highlight yaml %}
language: python
addons:
  firefox: latest
services:
  - postgresql
python:
  - "3.6"
before_install:
  - wget https://github.com/mozilla/geckodriver/releases/download/v0.18.0/geckodriver-v0.18.0-linux64.tar.gz
  - mkdir geckodriver
  - tar -xzf geckodriver-v0.18.0-linux64.tar.gz -C geckodriver
  - export PATH=$PATH:$PWD/geckodriver
  - geckodriver --version
  - export MOZ_HEADLESS=1
# command to install dependencies
install: pip install -r requirements.txt
# command to run tests
script: python manage.py test
{% endhighlight %}

Keep in mind that headless mode is supported in Firefox 56 on Windows and MacOS.
