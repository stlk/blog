# [Hyde](http://andhyde.com)

An elegant open source, mobile-first theme for [Jekyll](https://github.com/mojombo/jekyll). It includes lightweight styles and placeholder content to get you up and running with a simple blog in no time.

![Hyde screenshot](https://f.cloud.github.com/assets/98681/1330948/de10196c-353f-11e3-86d0-8e967dd95722.png)

Open sourced under the [MIT license](LICENSE.md).

<3


### Deploy

```
docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/minimal:latest rake site:publish
cd _site
git push origin master:refs/heads/gh-pages --force
```

### Development

```
docker run --rm --volume="$PWD:/srv/jekyll" -p 127.0.0.1:4000:4000 -it jekyll/minimal:latest jekyll serve --draft
```