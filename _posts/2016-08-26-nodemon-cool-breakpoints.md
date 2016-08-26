---
layout: post
title: "Nodemon: Breakpoints in node applications"
---

Nodemon is tool to improve backend JS development. It reloads files automatically so you don't have to restart the server every time. Another cool thing is that you can debug the code in chrome dev tools.

Lets take the express generated site as environment.

Fist we need to install nodemon.

{% highlight bash %}
$ npm i nodemon
{% endhighlight %}

Then run nodemon with `--inspect` parameter

{% highlight bash %}
$ node ./node_modules/nodemon/bin/nodemon.js --inspect ./bin/www
{% endhighlight %}

You can browse the code and even set breakpoints by clicking on the line number. This way you can inspect variables without having to use `console.log`.

<p class="post__image-center"><img src="/public/nodemon-folder.png" alt="How to get to source code" class="post__image-full-width"></p>

When you are done, click play icon to continue execution of a program.

<p class="post__image-center"><img src="/public/nodemon-continue.png" alt="How to continue program" class="post__image-full-width"></p>

Thanks [@mxstbr](https://twitter.com/mxstbr/status/768597725625454592) for bringing this up.
