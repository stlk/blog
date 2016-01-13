---
layout: post
title: Recommending Instagram photos using scikit-learn
---

I recently published first working version of my latest project. It is an Instagram newsletter based on pretty basic Machine Learning method. I called it [socialist](http://socialist.rousek.name). Feel free to try it out I would love to hear your feedback.

There were multiple motivations behind this project. I had strong urge to start Django project, create something related to Instagram, hopefully do some Machine Learning along the way, use git properly and finally use the most mainstream (maybe only in my bubble) tools specifically Github, Travis and Heroku. All this led to simple outcome. Start project as open source.

"Magic" hidden behind is summarized in two Jupyter notebooks. First one selects recent photos similar to what user has liked. Instagram doesn't allow much in its API, so I combined two queries and some nice images appeared.

Except sometimes totally unrelated image sneaked in. So I started looking for some primitive way to analyze distance between two users. It took me several weeks of occasional work to to find example I could use. I ended up with a great article [Text Analysis with Topic Models](https://www.de.dariah.eu/tatom/working_with_text.html). Yes, only text. My premise was that for basic distinction text analysis should be enough. And luckily it was.

To filter out posts from unrelated users I used following process. Download latest 200 photos for each users and throw them to [CountVectorizer](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html) to create a document-term matrix. Then use the matrix to filter media too distant from users.

You can find both notebooks and project itself on Github.com:

- [socialist](https://github.com/stlk/socialist)
- [photo recommendation](https://github.com/stlk/socialist/blob/master/jupyter/photo_recomendation.ipynb)
- [working with document-term matrix](https://github.com/stlk/datatalks/blob/master/instagram/mds.ipynb)
