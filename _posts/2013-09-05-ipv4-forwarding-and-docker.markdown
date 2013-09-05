---
layout: post
title:  "ipv4 Forwarding and Docker"
date:   2013-09-05 13:30:00
categories: ubuntu network docker
comments: true
---

Today I started playing with [Docker][docker] in earnest and ran into
a small problem that was easy to fix.  I don't often play with my network
so I had to look this one up.  I started by following the instructions for
installing [docker on my Mac][vagrant].  Then I went to the 
[introductory tutorial][getting_started] and followed along in real time.
I noticed as I was playing that I got this message:

{% highlight bash %}
WARNING: IPv4 forwarding is disabled.
{% endhighlight %}

Turns out that by default the ipv4 forwarding is not turned on in the
image from docker to prevent any security vulnerabilities.  I totally
get that but I wanted to turn it on and get rid of the message.  Here is
what I did:

{% highlight bash %}
$ sudo sysctl -w net.ipv4.ip_forward=1
{% endhighlight %}

Super simple solution.  But this won't work every time, you need to
update the actual /etc/sysctl.conf file to make it permanent.  Just
open up the file and uncomment the line with <code>net.ipv4.ip_forward=1</code>.
You're all done.  Exit and reload your vagrant image.  The error should
now disappear.

Read more about [this problem][ip_forwarding] if you're interested.

[docker]: http://docker.io
[vagrant]: http://docs.docker.io/en/latest/installation/vagrant/
[getting_started]: http://www.docker.io/gettingstarted
[ip_forwarding]: https://github.com/dotcloud/docker/issues/490
