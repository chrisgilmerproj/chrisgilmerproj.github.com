---
layout: post
title:  "Building Python with Pbuilder"
date:   2013-09-06 08:00:00
categories: debian python pbuilder semaphores
comments: true
---

Last night I noticed that my version of python built with pbuilder didn't
support the multiprocessing module or semaphore locks.  This came up
in a traceback during a code build:

{% highlight python %}
>>> from multiprocessing.synchronize import Lock
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python2.7/multiprocessing/synchronize.py", line 59, in <module>
    " function, see issue 3770.")
ImportError: This platform lacks a functioning sem_open implementation, therefore, the required synchronization primitives needed will not function, see issue 3770.
{% endhighlight %}

This was really strange behavior because I was certain that my debian systems
would support semaphores.  So I decided to look at the available information
in the given [issue number 3770][issue3770].  The issue essentially told me
that my build process didn't have semaphores enabled.  That's not surprising
because I'm building using pbuilder and I don't know if pbuilder correctly
mounts /dev/shm.  Looking at the build log I find this:

{% highlight bash %}
...
checking for sem_open... yes
checking for sem_timedwait... yes
checking for sem_getvalue... yes
checking for sem_unlink… yes
...
checking whether POSIX semaphores are enabled… no
{% endhighlight %}

Well that was informative.  The correct libraries are available as indicated
by the first part of that log snippet but the semaphores are still not enabled.
My first step was to test if /dev/shm was mounted on my build host:

{% highlight bash %}
$ mount -l | grep shm
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
{% endhighlight %}

Clearly it was there so why was it not in pbuilder?  To answer this question I
decided to log into pbuilder and see if /dev/shm was being mounted:

{% highlight bash %}
$  sudo pbuilder --login --configfile /etc/pbuilder/squeeze.pbrc 
I: Building the build Environment
I: extracting base tarball [/var/cache/pbuilder/squeeze-base.tgz]
I: creating local configuration
I: copying local configuration
I: mounting /proc filesystem
I: mounting /dev/pts filesystem
I: Mounting /var/cache/pbuilder/ccache
I: Mounting /var/cache/pbuilder/result
I: policy-rc.d already exists
I: Obtaining the cached apt archive contents
I: entering the shell
File extracted to: /srv/pbuilder/squeeze/5590

W: no hooks of type F found -- ignoring
root@tarbox:/# logout
I: Copying back the cached apt archive contents
I: unmounting /var/cache/pbuilder/result filesystem
I: unmounting /var/cache/pbuilder/ccache filesystem
I: unmounting dev/pts filesystem
I: unmounting proc filesystem
I: cleaning the build env 
I: removing directory /srv/pbuilder/squeeze/5590 and its subdirectories
{% endhighlight %}

Clearly pbuilder wasn't mounting /dev/shm.  So I read the man page and tried this:

{% highlight bash %}
$  sudo pbuilder --login --configfile /etc/pbuilder/squeeze.pbrc --bindmounts "/dev/shm"
I: Building the build Environment
I: extracting base tarball [/var/cache/pbuilder/squeeze-base.tgz]
I: creating local configuration
I: copying local configuration
I: mounting /proc filesystem
I: mounting /dev/pts filesystem
I: Mounting /dev/shm
I: Mounting /var/cache/pbuilder/ccache
I: Mounting /var/cache/pbuilder/result
I: policy-rc.d already exists
I: Obtaining the cached apt archive contents
I: entering the shell
File extracted to: /srv/pbuilder/squeeze/5590

W: no hooks of type F found -- ignoring
root@tarbox:/# logout
I: Copying back the cached apt archive contents
I: unmounting /var/cache/pbuilder/result filesystem
I: unmounting /var/cache/pbuilder/ccache filesystem
I: unmounting /dev/shm filesystem
I: unmounting dev/pts filesystem
I: unmounting proc filesystem
I: cleaning the build env 
I: removing directory /srv/pbuilder/squeeze/5590 and its subdirectories
{% endhighlight %}

This time /dev/shm was mounted and unmounted correctly.  With that in mind I
edited the config file:

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

In this case you can see that for <code>BINDMOUNTS</code> I provided a space
delimited list that includes /dev/shm.

Now I rebuild python with pbuilder and I get this output in the build log:

{% highlight bash %}
...
checking for sem_open... yes
checking for sem_timedwait... yes
checking for sem_getvalue... yes
checking for sem_unlink… yes
...
checking whether POSIX semaphores are enabled… yes
{% endhighlight %}

Success!  After installing I can run the same import again and I should get no
traceback:

{% highlight python %}
>>> from multiprocessing.synchronize import Lock
>>>
{% endhighlight %}

All fixed.  Now I can re-install python everywhere I had before and semaphores
will now work.

[issue3770]: http://bugs.python.org/issue3770 
