---
layout: post
title: Pipenv is great
---

I wanted to write something about Pipenv for a long time. With the new release, I think it's a good time to thank authors and maintainers of Pipfile. I started using Pipenv exactly two years ago and haven't stopped ever since. This post is a writeup of my views from the perspective of app developer.

### Alternatives, or not?

With the release of Django 3.0 Pipenv had a hard time correctly resolving asgiref package, I had to add it to the `Pipfile`. I thought this is where it ends so I started looking for alternatives.

#### venv + pip + pip-tools

This is just not for me. But it works for [others](https://twitter.com/AdamChainz/status/1239172374718808065).

#### Poetry

Poetry is intended for libraries. But we tried it anyway. However, it doesn't handle installing [specific](https://github.com/stlk/django-toolbox/) packages from git in editable mode. Thanks to [Test & Code podcast](https://testandcode.com/119) I discovered that editable installs are a feature of setuptools. I know the fault is [mine](https://github.com/stlk/django-toolbox/), but I just couldn't make it work.

#### Pyflow

I only heard about [Pyflow](https://github.com/David-OConnor/pyflow) in [](https://pythonbytes.fm/episodes/show/186/the-treebeard-will-guard-your-notebook) but it looks promising. The nice thing is that it implements proposals that will most likely make it into python itself. It's ticking all the boxes for me so I might give it a shot.

### Still using it

Overall Pipenv is the perfect tool for my workflow. Version pinning, loading ENV variables, automatic environment creation and activation are just the chores I don't want to worry about. I believe that package development and application development are so different tasks that I'm fine with Pipenv not adopting `pyproject.toml`. However, I expect that prominent community members communicate what tool should be used for application development. From the outside, it looks like everybody is concerned about packages. Yes, making package development simple is an important goal. But in the end, most of us are just users. We're building applications. That's why I love Pipenv.

