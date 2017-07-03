---
layout: post
title: Next.js - server side rendering for masses
---

Recently I had opportunity to use [Next.js](https://github.com/zeit/next.js) on a client project. I already heard how much work it saves but I just couldn't believe how simple it was.

My requirements were following. Server rendered react application with jest as testing framework and [styled-components](https://www.styled-components.com) as the *best* way to manage styles. If you aren't interested in these be sure to checkout [blogpost announcing version 2.0](https://zeit.co/blog/next2#more-examples) of Next.js. There's a bunch of examples showing integrations with other libraries.

To grasp the basics I followed tutorial on [Learning Next.js](https://learnnextjs.com). It got me through some early surprises. Eg. how route resolving is done on the server when using custom routes. But first lets take a look on how you define custom route. Magic is hidden behind the `as` attribute and everything is handled automatically.

{% highlight javascript %}
<Link as={`/post/${show.id}`} href={`/post?id=${show.id}`}>
  <a>{show.name}</a>
</Link>
{% endhighlight %}

On the other hand server configuration is done on, well, server level. We need to manually define a route and then tell `next.js` which page is it. You can use plethora of nodejs servers as backend, but I've used the default one which is `express`.

{% highlight javascript %}
server.get('/post/:id', (req, res) => {
  const actualPage = '/post';
  const queryParams = { id: req.params.id };
  app.render(req, res, actualPage, queryParams);
});
{% endhighlight %}

Please note that this is required only for server side rendering with custom routes.

### Styled components

First library I want to integrate is [styled-components](https://github.com/zeit/next.js/tree/master/examples/with-styled-components) integrating this library is pretty simple. Just copy `pages/_document.js` from example repo. To not break server side rendering we need to use babel plugin called `babel-plugin-styled-components`. Don't forget to include `.babelrc`, I prefer to keep my babel configuration in `package.json`.

{% highlight json %}
{
  "name": "hello-next",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "babel": {
    "presets": [
      "next/babel"
    ],
    "plugins": [
      [
        "styled-components",
        {
          "ssr": true
        }
      ]
    ]
  },
  "scripts": {
    "dev": "node server.js",
...
{% endhighlight %}

### Tests using jest

To set up jest all you need to do is to add `env` to [babel presets](https://github.com/stlk/next-workshop/blob/master/package.json#L33).

I also have a [script](https://github.com/stlk/next-workshop/blob/master/test.js) that will automatically disable `watch` when run on CI server.

### Want to learn more?

You can follow my progress on [GitHub.com](https://github.com/stlk/next-workshop).

### Školení Next.js

Vedu [Next.js workshop](https://react.coffee/#react-na-serveru), kde si vyzkoušíme Next.js v praxi.
