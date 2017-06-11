---
layout: post
title: Automatically format your javascript files on commit using Prettier
---

I recently fell in love with [prettier](https://github.com/prettier/prettier). Prettier is an opinionated code formatter.
Having consistently formatted code is very pleasant feeling for me and having formatting done automatically?
Woah! The best part is that you can easily run `prettier` on `git commit`. That way you can write messy code and it will get cleaned up automatically. Let's do it.

By combining three packages we can create pre-commit git hook that will transform javascript files using [prettier](https://github.com/prettier/prettier). Those packages are `lint-staged`, `pre-commit` and `prettier`.
To install them run `npm install --save-dev lint-staged pre-commit prettier`.

Then add following keys to your `package.json`.

{% highlight json %}
{
  "scripts": {
    "lint:staged": "lint-staged"
  },
  "lint-staged": {
    "*.js": [
      "prettier --write --single-quote --trailing-comma=all",
      "git add"
    ]
  },
  "pre-commit": "lint:staged"
}
{% endhighlight %}

Now when you commit you code you should see `✔ Running tasks for *.js` message.

{% highlight bash %}
$ git commit -m 'Initial commit'
 ✔ Running tasks for *.js
[master (root-commit) 7da3cc3] Initial commit
 13 files changed, 4841 insertions(+)
{% endhighlight %}
