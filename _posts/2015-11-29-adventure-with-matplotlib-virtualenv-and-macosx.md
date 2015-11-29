---
layout: post
title: Adventure with matplotlib, virtualenv and MacOS
---

Recently I've developed passion for machine learning. Which includes many hours of fun with various modeling and plotting libraries.

Yesterday I was playing around with [Non-Negative Matrix Factorization (NMF)](http://scikit-learn.org/stable/modules/generated/sklearn.decomposition.NMF.html) and [Multidimensional scaling](http://scikit-learn.org/stable/modules/generated/sklearn.manifold.MDS.html) in jupyter notebook. But when I tried to render a chart it just didn't work.
Engine got stuck and I had to restart it. At first I though that the rendering is too complicated, but even simple example didn't work.

Problem was as I later discovered caused by a [bug](https://github.com/pypa/virtualenv/issues/54) in virtualenv. I managed to get it working using workaround described on [matplotlib's FAQ](http://matplotlib.org/devdocs/faq/virtualenv_faq.html#osx).

First install python and create virtualenv.

{% highlight sh %}
brew install python --framework # install python 2.7 framework build
virtualenv env # create virtualenv
{% endhighlight %}

Then create bash script called `frameworkpython` in `bin` folder of your environment, in my case it's `env/bin/frameworkpython` with following content.

{% highlight sh %}
#!/bin/bash

# what real Python executable to use
PYVER=2.7
PATHTOPYTHON=/usr/local/bin/
PYTHON=${PATHTOPYTHON}python${PYVER}

# find the root of the virtualenv, it should be the parent of the dir this script is in
ENV=`$PYTHON -c "import os; print os.path.abspath(os.path.join(os.path.dirname(\"$0\"), '..'))"`

# now run Python with the virtualenv set as Python's HOME
export PYTHONHOME=$ENV
exec $PYTHON "$@"
{% endhighlight %}

Make the scipt executable and install libraries.

{% highlight sh %}
chmod +x env/bin/frameworkpython # make it executable
source env/bin/activate # activate environment
pip install matplotlib jupyter notebook # install matplotlib and jupyter notebook
{% endhighlight %}

Only one problem was ahed of me. How to start the notebook from this script. It was pretty simple in the end `frameworkpython env/bin/jupyter-notebook`.

You can use this code to test it. Don't forget to use `%matplotlib inline` in your notebook to prevent window popping up.

{% highlight python %}
%matplotlib inline
import matplotlib.pyplot as plt
import numpy as np

t = np.arange(0.0, 2.0, 0.01)
s = np.sin(2*np.pi*t)
plt.plot(t, s)

plt.grid(True)
plt.show()
{% endhighlight %}

I hope this article will save you some hours of your life I spent with this. You can see the result on my [github](https://github.com/stlk/datatalks/blob/master/instagram/nmf_mds.ipynb).
