---
layout: post
title:  "Starting with Python 3"
date:   2013-07-04 10:30:00
categories: python pip virtualenv
comments: true
---

It's finally time to get started with python 3.  I've been putting it off for a
while because of concerns that the libraries are not yet ready, that my code
would require a lot of massaging, and pretty much because I dislike changing
my practices.  I like to make small changes instead of throwing the big switch
and digging through lots and lots of problems.

Well after running into python 3 code at work and attending PyCon 2013 back in
March I'm pretty convinced I just need to charge ahead.  Fortunately for me it
couldn't be easier to set up python 3 and to get started.

I use [homebrew][homebrew] for package management on my mac.  I love it because
it gives me access to all the great utilities I'm used to from Ubuntu and Debian
with the advantages of having a mac as my personal machine.  For me getting
started was easy:

{% highlight bash %}
$ brew update
$ brew install python3
==> Downloading http://python.org/ftp/python/3.3.2/Python-3.3.2.tar.bz2
Already downloaded: /Library/Caches/Homebrew/python3-3.3.2.tar.bz2
==> ./configure --prefix=/usr/local/Cellar/python3/3.3.2 --enable-ipv6 --datarootdir=/usr/local/Cellar/python3/3.3.2/share --datadir=/usr/local/Cellar/python3/3.3.2/share --enable-fr
==> make
==> make install PYTHONAPPSDIR=/usr/local/Cellar/python3/3.3.2
==> make frameworkinstallextras PYTHONAPPSDIR=/usr/local/Cellar/python3/3.3.2/share/python3
==> Downloading https://pypi.python.org/packages/source/d/distribute/distribute-0.6.45.tar.gz
Already downloaded: /Library/Caches/Homebrew/distribute-0.6.45.tar.gz
==> /usr/local/Cellar/python3/3.3.2/bin/python3.3 -s setup.py install --force --verbose --install-scripts=/usr/local/Cellar/python3/3.3.2/bin --install-lib=/usr/local/lib/python3.3/s
==> Downloading https://pypi.python.org/packages/source/p/pip/pip-1.3.1.tar.gz
Already downloaded: /Library/Caches/Homebrew/pip-1.3.1.tar.gz
==> /usr/local/Cellar/python3/3.3.2/bin/python3.3 -s setup.py install --force --verbose --install-scripts=/usr/local/Cellar/python3/3.3.2/bin --install-lib=/usr/local/lib/python3.3/s
==> Caveats
Distribute and Pip have been installed. To update them
  pip3 install --upgrade distribute
  pip3 install --upgrade pip

To symlink "Idle 3" and the "Python Launcher 3" to ~/Applications
  `brew linkapps`

You can install Python packages with
  `pip3 install <your_favorite_package>`

They will install into the site-package directory
  /usr/local/lib/python3.3/site-packages

See: https://github.com/mxcl/homebrew/wiki/Homebrew-and-Python
==> Summary
/usr/local/Cellar/python3/3.3.2: 4699 files, 92M, built in 4.3 minutes
{% endhighlight %}

The next thing you probably want to do is try it out with a current project of
yours.  This is pretty easy if you head over to your project and create a
virtual environment.  I'll show you two different ways.  Assuming you have
virtualenv already installed you can run this command which will correctly
use python 3:

{% highlight bash %}
$ virtualenv env3 --python `which python3`
Using base prefix '/usr/local/Cellar/python3/3.3.2/Frameworks/Python.framework/Versions/3.3'
New python executable in env/bin/python3.3
Also creating executable in env/bin/python
Installing distribute.........done.
Installing pip................done.
{% endhighlight %}

The second and probably more straight forward way is simply to install a python 3
version of virtualenv.  To do this you'll end up using <code>easy_install</code>

{% highlight bash %}
$ easy_install-3.3 virtualenv
Searching for virtualenv
Reading http://pypi.python.org/simple/virtualenv/
Best match: virtualenv 1.9.1
Downloading https://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.9.1.tar.gz#md5=07e09df0adfca0b2d487e39a4bf2270a
Processing virtualenv-1.9.1.tar.gz
Writing /var/folders/wt/w5r4g27579n28673yx1ftjyr0000gn/T/easy_install-tt0yg7/virtualenv-1.9.1/setup.cfg
Running virtualenv-1.9.1/setup.py -q bdist_egg --dist-dir /var/folders/wt/w5r4g27579n28673yx1ftjyr0000gn/T/easy_install-tt0yg7/virtualenv-1.9.1/egg-dist-tmp-66fy1m
warning: no previously-included files matching '*' found under directory 'docs/_templates'
warning: no previously-included files matching '*' found under directory 'docs/_build'
Adding virtualenv 1.9.1 to easy-install.pth file
Installing virtualenv-3.3 script to /usr/local/bin
Installing virtualenv script to /usr/local/bin

Installed /usr/local/lib/python3.3/site-packages/virtualenv-1.9.1-py3.3.egg
Processing dependencies for virtualenv
Finished processing dependencies for virtualenv
$ virtualenv-3.3 env3
Using base prefix '/usr/local/Cellar/python3/3.3.2/Frameworks/Python.framework/Versions/3.3'
New python executable in env/bin/python3.3
Also creating executable in env/bin/python
Installing distribute.........done.
Installing pip................done.
{% endhighlight %}

Next you should activate the virtualenv and check the python version:

{% highlight bash %}
$ source env3/bin/activate
(env3) $ python --version
Python 3.3.2
{% endhighlight %}

One thing that might trip you up.  When homebrew installed python 3 for you
it helpfully makes a <code>pip3</code> version for you.  This is great if you're
using <code>pip3</code> to install python packages globally.  What might not be
apparent is that inside your virtualenv the same executable is called 
<code>pip</code>.  Since you are in a virtualenv you only need one version of
it available, but in the global space where you have both python 2 & 3 installed
its nice to be able to differentiate.  Don't worry, <code>pip</code> will indeed
install python 3 related packages to your python 3 installation in your
environment.

Now try to install something:

{% highlight bash %}
$ pip install ipython
...
$ pip freeze > requirements.txt
$ cat requirements.txt
distribute==0.6.34
ipython==0.13.2
{% endhighlight %}

In the future you can use that requirements.txt file to install packages:

{% highlight bash %}
$ pip install -r requirements.txt
{% endhighlight %}

This is a great way to version control all your package dependencies and help
others install them when they check out your code.

Good luck with python 3 and I hope this article helped.

[homebrew]: http://mxcl.github.io/homebrew/ 
