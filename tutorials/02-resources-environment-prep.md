#UFO Data Science - Resources and Environment Preparation

Though I’ve been focused on building a Django website for the time being, I at least have an outline of the next steps to take with the data pulled from [NUFORC](http://www.nuforc.org/). (Hint: It involves [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/bs4/doc/) and [MongoDB](http://api.mongodb.org/python/current/index.html) workflow.) This post is really a footnote of setup information needed to simply have in place before going forward.

First of all, I’ve got a couple key learning resources queued up for the big chunks of knowledge I will be taking in:

1. [Kevin Sheppard’s *Introduction to Python for Econometrics, Statistics, and Data Analysis*](http://www.kevinsheppard.com/images/0/09/Python_introduction.pdf), hosted for free on his site. This looks as though it will be a primary resource, seeing as it outlines exercises and workflows done via the Anaconda scientific Python package collection.
2. In hard copy, I have Wes McKinney’s [*Python for Data Analysis*](http://shop.oreilly.com/product/0636920023784.do) from O’Reilly as a good secondary resource on the topic.
3. Additionally, I’ll be trying to interact regularly with the relevant lessons coming from a couple Udacity courses, [Intro to Computer Science](https://www.udacity.com/course/cs101) and [Intro to Data Science](https://www.udacity.com/course/ud359).

For the environment itself, [Anaconda is a fairly straightforward installation](http://docs.continuum.io/anaconda/install.html). Once completed, we can quickly have ourselves set up for Anaconda work with a few commands, provided by Sheppard’s introduction:

```
$ conda update conda
$ conda update anaconda
$ pip install pylint html5lib seaborn
```
*This last command also installs ```logilab-common``` and ```astroid```, two dependent packages.*

Next, we can confirm the IPython (```$```), NumPy(```ln [1-2]```), matplotlib (```ln [3]```), and SciPy (```ln [4]```) installations, if all the following commands successfully run:

```
$ ipython qtconsole --pylab
ln [1]: x = randn(100,100)
ln [2]: y = mean(x,0)
ln [3]: plot(y)
ln [4]: import scipy as sp
```

And finally, we install the MathJax JavaScript library for LaTeX math formatting within IPython:

```
$ ipython
ln [1]: from IPython.external.mathjax import install_mathjax
ln [2]: install_mathjax()
ln [3]: exit
```

All set, up and running! That’s it for now.

[Link to original post](http://scotterenaissance.tumblr.com/post/97137337033/ufo-data-science-resources-and-environment)
