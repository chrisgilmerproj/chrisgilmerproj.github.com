---
layout: post
title:  "Mirror a Debian Repo"
date:   2013-08-29 15:00:00
categories: debian mirror rsync
comments: true
---

You may decide, like my company, that mirroring the official debian repo is a
good idea for a lot of good reasons.  A really good reason is to ensure that
you always have access to these packages even when you cannot connect to the
official mirror.  It turns out this is really quite easy to do with rsync and
this post will show you how.

The first thing you want to read before you get started is to read the article
on [Setting up a Debian archive mirror][mirror].  It's very good and gives you
several options in case rsync doesn't work for you.

To get started you want to first figure out which mirror you want to mirror.
Using rsync is really easy and you can quickly set up a full mirror script in
two steps:

{% highlight bash %}
set -e
set +o noglob
FLAGS="-H -a --no-motd"
OPTIONS1="--exclude Packages* \
          --exclude Sources* \ 
          --exclude Release* \ 
          --exclude InRelease \
          --exclude i18n/* \   
          --exclude ls-lR* \   
          --exclude .~tmp~"    
OPTIONS2="--delete-after" 
/usr/bin/rsync $FLAGS $OPTIONS1 ftp.us.debian.org::debian/ /srv/debian &&
/usr/bin/rsync $FLAGS $OPTIONS2 ftp.us.debian.org::debian/ /srv/debian &&
/bin/date -u > /srv/debian/project/trace/`hostname`
{% endhighlight %}

That's it!  You can put that in a script, run it with cron, and serve up the 
files with apache2.  But let's pull this apart so that you know what it means.

First you set some flags:

{% highlight bash %}
set -e
set +o noglob
{% endhighlight %}

The <code>set -e</code> flag is used to ensure that the script stops if there
are any errors during execution.  This is super helpful to stop the script from
continuing onto following steps if any previous step breaks.  The
<code>set +o noglob</code> is used to ensure bash doesn't have fun with the *
characters inside your code. The * characters are supposed to be for rsync
to glob match patterns.  However, if you have any files in your script's
folder that match one of these patterns then bash will "helpfully" expand
them for you.  This will cause you all sorts of errors.  So use these two
flags to make sure life is easy for you.

Next you'll notice that <code>FLAGS="-H -a --no-motd"</code> is included.
This is an easy way to set up the rsync flags in one place so they can be
used in every rsync command.  You can read the man page on the flags but
importantly it's useful to include <code>--no-motd</code> so that your
script doesn't get a lot of output from the debian server that will get
logged in during your cron's run.

Finally the rsync is done in two steps.  The first stage syncs up all of the
files so that you have a up-to-date copy of the mirror.  The second stage
copies over all of the files excluded in the first stage.  These are the
files that tell apt what is available from the mirror.  If you choose to do
this all in one step these special files may reference files that don't yet
exist yet.  That will cause a problem with any hosts that are running
<code>apt-get update</code> during the run of your mirror script.

Once all the files have been copied over there is a <code>--delete-after</code>
flag that will remove any files that are no longer on the debian mirror
after everything is synced up.

You may have noticed there is an extra step at the end.  If you read the docs
you'll notice that it recommends you add a file in the mirror under
/project/trace named after your mirror host.  If your hostname is 'wookie'
then your file would be /project/trace/wookie and the contents would be from
<code>/bin/date -u</code>.

If you decide you don't need all the architectures copied over to your internal
mirror you can exclude those too.  This will speed up the time it takes to sync
up, making your devs happier.  In my case I only want amd64 and source so I 
enumerate and exclude everything else.  An easy way to do this is this:

{% highlight bash %}
EXCLUDE_ARCH="alpha arm"    
EXCLUDE=""
for arch in $EXCLUDE_ARCH; do  
     EXCLUDE="--exclude arch-${arch}* \
              --exclude binary-${arch}/ \
              --exclude *_${arch}.deb \
              --exclude *_${arch}.udeb \
              --exclude installer-${arch}/ \
              --exclude Contents-${arch}* \
              --exclude Contents-udeb-${arch}* \
              $EXCLUDE"        
done
{% endhighlight %}

Add this into the script above and you're set.  Note that I don't enumerate the
architectures here for you because they may be different when you read this
post.

Good luck with mirroring the Debian repo for your own use.


[mirror]: http://www.debian.org/mirror/ftpmirror
