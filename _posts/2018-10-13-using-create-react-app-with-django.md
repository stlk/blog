---
layout: post
title: Using Create React App 2 with Django 2
image: http://blog.rousek.name/public/using-create-react-app-with-django.png
---

I finally got opportunity to work on a project that used Django together with a React app. There's a combination of packages that help with integrating CRA and Django.

### webpack-bundle-tracker + django-webpack-loader

`django-webpack-loader` and `webpack-bundle-tracker` work together in the following way. webpack-bundle-tracker plugin emits information about webpack’s compilation process and results so django-webpack-loader can consume it. To add `webpack-bundle-tracker` to CRA we will need to hook up a plugin to webpack.

### CRA configuration

With your CRA app generated we start with `react-app-rewired` configuration. Rewired uses `config-overrides.js` file to hook into webpack configuration. There is one more change we have to make apart from adding `BundleTracker`. The new [splitChunks](https://medium.com/webpack/webpack-4-code-splitting-chunk-graph-and-the-splitchunks-optimization-be739a861366) configuration is not entirely supported, so we have to use only named chunks.

```js
var BundleTracker = require('webpack-bundle-tracker');

module.exports = {
  webpack: (config, env) => {

    config.plugins.push(
      new BundleTracker({
        path: __dirname,
        filename: './build/webpack-stats.json',
      }),
    );

    config.optimization.splitChunks.name = 'vendors';

    return config;
  },
};
```

After running `npm start` we should see additional file `webpack-stats.json` in build folder containing all the data we need.

### Django configuration

Install `django-webpack-loader` using [pipenv](https://packaging.python.org/tutorials/managing-dependencies/) or pip. Then go to your `settings.py`. Add webpack_loader to INSTALLED_APPS. This will enable us to use template tags.

```py
INSTALLED_APPS = [
    ...
    'webpack_loader',
]
```

#### Project structure

My project directory structure looks like this. I decided to put my CRA app to `frontend` folder.

```tree
├── Pipfile
├── Pipfile.lock
├── README.md
├── my_django_app
│   ├── __init__.py
│   ├── context_processors.py
│   ├── settings.py
│   ├── urls.py
│   ├── utils.py
│   └── wsgi.py
├── frontend
│   ├── build
│   ├── config-overrides.js
│   ├── node_modules
│   ├── package-lock.json
│   ├── package.json
│   ├── public
│   └── src
└── manage.py
```

With following structure, we can configure webpack loading in following way.

```py
STATICFILES_DIRS = [
    # So Django knows about webpack files
    os.path.join(BASE_DIR, "frontend", "build", "static"),
]

WEBPACK_LOADER = {
    "DEFAULT": {
        "CACHE": not DEBUG,
        "BUNDLE_DIR_NAME": "frontend/build/static/",  # must end with slash
        "STATS_FILE": os.path.join(BASE_DIR, "frontend", "build", "webpack-stats.json"),
    }
}
```

### Usage

In our Django template we can then use `render_bundle` to load our CRA app.

```liquid
{% raw %}
{% load render_bundle from webpack_loader %}
{% render_bundle 'vendors' %}
{% render_bundle 'main' %}
{% render_bundle 'runtime~main' %}
{% endraw %}
```

It will also display the webpack error when the build process is failing.

<p class="post__image-center">
  <img src="/public/django-webpack-error.png" alt="Webpack error in Django" class="post__image post__image-full-width">
</p>

I was very happy about getting Django and CRA to work together in this way and I can only recommend it.

#### Projects

- [django-webpack-loader](https://github.com/owais/django-webpack-loader)
- [webpack-bundle-tracker](https://github.com/owais/webpack-bundle-tracker)