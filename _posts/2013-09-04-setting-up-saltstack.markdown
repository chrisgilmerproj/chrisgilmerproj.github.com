---
layout: post
title:  "Setting Up Saltstack"
date:   2013-09-04 12:00:00
categories: debian python saltstack
comments: true
---

Lately I've been working with [SaltStack][saltstack] and I have a few 
easy-to-implement items that might make your life easier when setting
up SaltStack on your own set of hosts.  I'm going to go through both the
salt-master and the salt-minion configuration to get up and running
and from there you can get to customizing your own installation.

If you don't have a user and group named <code>salt:salt</code> for your hosts
you will have to make them with a script:

{% highlight bash %}
#!/bin/bash

SLT_USER="salt"
SLT_GROUP="salt"
if [ ! /usr/bin/getent group "$SLT_GROUP" > /dev/null 2>&1 ]; then
    sudo /usr/sbin/addgroup --system "$SLT_GROUP" --quiet
fi
if [ ! /usr/bin/id $SLT_USER > /dev/null 2>&1 ]; then
    sudo /usr/sbin/adduser --system --home /home/salt \
    --ingroup "$SLT_GROUP" --disabled-password --shell /bin/bash \
    "$SLT_USER"
fi
{% endhighlight %}

Once you've done this you can install the salt-master and salt-minion
on the appropriate hosts of your choosing.

Configuring the salt-master:
----------------------------

Let's start off with the salt-master.  If you're just getting started with
you will only have one salt-master.  This is typical of installation and
if you want to know more about multi-master setups you can read the docs.

The first step is to edit the defaults file that your init.d script will
use when setting up your salt-master daemon.  Create or edit a file in
/etc/default/salt-master:

{% highlight bash %}
DAEMON_ARGS="-d -u salt"
PATH=/usr/local/bin:$PATH  
{% endhighlight %}

The <code>DAEMON_ARGS</code> indicates that you want to run the master
in daemon-mode and to use the user named 'salt' to run it.  The
<code>PATH</code> simply adds <code>/usr/local/bin</code> because otherwise
your salt environment won't be able to find or run any scripts or binaries
from that location. You can add more if you'd like.

The second thing you'll do for the salt-master is create or edit a file
in /etc/salt/master.d/standard.conf:

{% highlight yaml %}
user: salt
{% endhighlight %}

Any files with the \*.conf extension in the /etc/salt/master.d directory will be
read into the configuration of the salt-master.  This is nice because then you
won't have to edit the default salt-master template that comes with the
installation.  I find that this is super helpful because you don't want to have
to version control that default template file and along with the standard.conf
file you can always provide a local.conf file for any local changes to a
particular host.

Simply restart your salt-master and move onto managing the minion:

{% highlight bash %}
$ sudo /etc/init.d/salt-master restart
{% endhighlight %}

Configuring the salt-minion:
----------------------------

The next thing we'll do is configure our salt-minion.  This is almost identical
to the salt-master config.  The first thing we'll edit is the
/etc/default/salt-minion file.  This will be IDENTICAL to the file in
/etc/default/salt-master.  Again, this just sets defaults we want for our init.d
script.

The second file is similar in location, here we'll create or edit a file named
/etc/salt/minion.d/standard.conf:

{% highlight yaml %}
master: salt
{% endhighlight %}

Now I know by default that the salt-master is normally located on a host named
'salt', but in case you change that location you can edit it in this file.

Commonly you want the id of the host to not be the fully qualified domain
name (fqdn) which is used by default.  To fix this you can do the following
on each of your minions:

{% highlight bash %}
$ HOST=$( hostname -s )      
$ echo -e "id: $HOST" > /etc/salt/minion.d/local.conf
{% endhighlight %}

This will ensure that your host name is used instead of the fqdn for targetting.

Now restart your salt-minion:

{% highlight bash %}
$ sudo /etc/init.d/salt-minion restart
{% endhighlight %}

The last piece is easy, you need to accept the salt-minion key from the master.
Follow the steps in the docs for this.  I won't repeat them as they may change.
Finally, restart your minion one more time and everything should work.  Try
out some test commands and the targeting by hostname and everything should work.

[saltstack]: http://docs.saltstack.com/
