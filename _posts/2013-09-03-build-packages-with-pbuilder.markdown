---
layout: post
title:  "Build packages with pbuilder"
date:   2013-09-03 11:00:00
categories: debian packages pbuilder
comments: true
---

A really good practice is to build debian packages inside the environment you
intend to deploy in.  For example, if you plan to deploy your package on an
amd64 wheezy host then you ought to build and test your package install on a
amd64 wheezy host.  Seems pretty straight forward, eh?  But lots of people miss
this when building packages for debian.  I'm going to go over using a tool
called pbuilder to do exactly this.

After you install pbuilder you will need to do a couple things.
First we need to do some setup for our host:

{% highlight bash %}
$ mkdir -p /etc/pbuilder/repo/
$ mkdir -p /srv/pbuilder/wheezy/
$ mkdir -p /var/cache/pbuilder/result/wheezy/
$ touch /var/cache/pbuilder/result/wheezy/Packages
$ chmod 666 /var/cache/pbuilder/result/wheezy/Packages
$ /usr/sbin/pbuilder --create --configfile /etc/pbuilder/repo/wheezy.pbrc > /var/cache/pbuilder/wheezy-base.log
{% endhighlight %}

Then we need to make a file named wheezy.pbrc in the /etc/pbuilder/repo/ directory:

{% highlight bash %}
MIRRORSITE=http://myhost/debian/
DISTRIBUTION=wheezy
COMPONENTS="main contrib non-free"
BASETGZ=/var/cache/pbuilder/wheezy-base.tgz
BINDMOUNTS="/var/cache/pbuilder/result /dev/shm"
HOOKDIR="/var/cache/pbuilder/hooks"
BUILDPLACE="/srv/pbuilder/wheezy"
OTHERMIRROR="deb file:/var/cache/pbuilder/result/wheezy /|deb http://myhost/external/ wheezy main backports"
APTCACHEHARDLINK=no
{% endhighlight %}

You'll notice that I point <code>MIRRORSITE</code> to the my debian mirror and
<code>OTHERMIRROR</code> to the external mirror and the build results from
wheezy.  Most of the other settings are important but don't need to be covered
here.

Next I'm going to assume you have a set of three files that together represent
your debian source package:

- mypackage.dsc
- mypackage_source.changes
- mypackage.tar.gz

These are the necessary files required to build your debian package.  You will
now run the following from the directory where your debian source files reside:

{% highlight bash %}
$ PACKAGE=mypackage.dsc
$ TMPDIR=`mktemp -d`
$ sudo /usr/sbin/pbuilder --build --buildresult ${TMPDIR} --configfile /etc/pbuilder/repo/wheezy.pbrc ${PACKAGE} > ${TMPDIR}/build.log 
{% endhighlight %}

You should now have three additional files in the <code>${TMPDIR}</code> directory
that you specified:

- mypackage_all.deb
- mypackage_amd64.changes
= build.log

The mypackage_all.deb file is the file you will install on your debian host. If
you are curious about the build process you can inspect the build.log file for
more information.

Now what do you do if your build fails and you want to drop in as root to
see what the problem is?  This is where your <code>HOOKDIR</code> comes in
handy.  You will put a file named C10shell in the /var/cache/pbuilder/hooks
directory that will drop you in as the root user if there is an error.
The following recipe from the pbuilder site will do that:

{% highlight bash %}
#!/bin/bash

echo "Installing helper packages..."       
apt-get install -y "${APTGETOPT[@]}" vim less 
echo "Invoking shell..."       
cd /tmp/buildd/*/debian/..     
/bin/bash < /dev/tty > /dev/tty 2> /dev/tty 
{% endhighlight %}

Hooks are a great way to get some extra functionality during build time like
adding new apt-keys or enabling extra repo's during build-time.

With all these tools you should now be able to build your debian source package
inside a clean, stable environment that you will deploy to making your whole
deployment chain better.
