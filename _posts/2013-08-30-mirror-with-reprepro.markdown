---
layout: post
title:  "Mirror with reprepro"
date:   2013-08-29 15:00:00
categories: debian mirror reprepro
comments: true
---

To follow up on my post about [Debian Mirrors][debian_mirrors]
I thought I'd do one on mirroring an external repo using a really good tool
called reprepro.  Using reprepro comes in handy for two operations.  The first 
is when you want to host your own internal repo of packages you've created.  
The second reason, and the one I'll cover here, is when you want to mirror a 
repo that you can't easily connect to via an rsync server.

Setting up reprepro is incredibly easy to do.  The first think you want to do is
install the package on your debian host.  The second thing you need to do is set
up a set of configuration files.  As an example we're going to do this for the
[debian SaltStack mirror][debian_saltstack].

On your host make a location for your repo. 

{% highlight bash %}
$ mkdir -p /srv/repos/external/{,conf}
{% endhighlight %}

Now you need to make three files and put them in the <code>/srv/repos/external/conf</code>
directory.  These files are named <code>distributions, options, and updates</code>.

The <code>/srv/repos/external/conf/distributions</code> file looks like this:

{% highlight bash %}
Origin: Saltstack              
Label: Saltstack
Suite: wheezy-saltstack
Codename: wheezy-saltstack     
Description: Saltstack nightly packages for wheezy
Architectures: amd64 source    
Components: main
SignWith: yes
Update: wheezy-saltstack       
Log: /var/log/reprepro/saltstack.log 
{% endhighlight %}

Let me pull this apart for you.  The important things here are the
<code>Architectures, Components, and SignWith</code>.  The first part you modify
for your specific machine.  In this case amd64 is the architecture of all
my hosts.  The next part is the available components and for SaltStack there
is only <code>main</code>.  You also need to make sure you use <code>SignWith</code>
so that you can get the correct Release.gpg file from the remote server.

The <code>/srv/repos/external/conf/updates</code> file looks like this:

{% highlight bash %}
Name: wheezy-saltstack
Method: http://debian.saltstack.com/debian
Suite: wheezy-saltstack        
Components: main
Architectures: amd64 source
VerifyRelease: B09E40B0F2AE6AB9
{% endhighlight %}

The <code>updates</code> file is used to grab the files from the remote
server.  This is pretty easy to understand because there are a lot of
similarities to the first one.  Where <code>distributions</code> is used
to server the repo files, <code>updates</code> is used to get the files
from the remote server.  The important part here is the
<code>VerifyRelease</code> part which I'll talk about in a little bit.

The <code>/srv/repos/external/conf/options</code> file looks like this:

{% highlight bash %}
basedir /srv/repos/external
gnupghome /srv/repos/.gnupg
keepunreferencedfiles
{% endhighlight %}

Finally the <code>options</code> file lets you set options that you
would otherwise have to include on the command line when calling
reprepro.  I've included three things here, the base directory, the
location of the gpg files, and I ensure that any time a new file is
added the old files are not deleted.

So this is pretty easy.  You can read the man page to learn more but
frankly this is the minimum subset to get going.  Next you need to
download the key to your gpg directory so you can verify that the
packages you're getting are indeed the ones you asked for.  This is
just standard security.  Follow this procedure:

{% highlight bash %}
$ HOMEDIR=/srv/repos/.gnupg
$ URL="http://debian.saltstack.com/debian/dists/wheezy-saltstack/"
$ wget ${URL}/Release.gpg
$ wget ${URL}/Release
$ gpg Release.gpg
$ echo "Enter the key that you want to search for, followed by [ENTER]:"
$ read KEY
$ gpg --homedir ${HOMEDIR} --keyserver subkeys.pgp.net --search-keys ${KEY}
$ gpg --homedir ${HOMEDIR} --with-colons --list-key
{% endhighlight %}

You can probably script that up but basically you're getting the Release
files, you get the key signature, you search for the key on the keyserver,
and finally add it to your keyring.  This is where you get the key
for the <code>VerifyRelease</code> in the <code>updates</code> file.

Now all you have to do is update the repo.  Run these commands:

{% highlight bash %}
$ /usr/bin/reprepro -b /srv/repos/external update
$ /usr/bin/reprepro -b /srv/repos/external export
{% endhighlight %}

Now add this to your <code>/etc/apt/sources.list</code> file:

{% highlight bash %}
deb http://myhost/external/ wheezy-saltstack main
{% endhighlight %}

Serve the files with apache2 and then update apt and you're good to go!

If you want to learn more you can always read the main [reprepro][reprepro]
page or the reprepro man page.  I also highly recommend the 
[IRC channel][freenode] #reprepro.  You can usually get help there for
any obscure questions.

[debian_mirrors]: {% post_url 2013-08-29-mirror-debian %}
[debian_saltstack]: http://debian.saltstack.com/
[reprepro]: http://mirrorer.alioth.debian.org/
[freenode]: http://irc.freenode.net
