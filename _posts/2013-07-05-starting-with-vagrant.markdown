---
layout: post
title:  "Starting with Vagrant"
date:   2013-07-05 09:00:00
categories: vagrant apache
comments: true
---

I'm moving over my development to use the environment into which I intend to
deploy.  For most cases this is going to be using some flavor of [Ubuntu][ubuntu]
which you can deploy on most servers.  Since my development machine is a
Mac Pro I'm going to use [VirtualBox][virtualbox] and [Vagrant][vagrant] to set
up this new environment.

The first thing you want to do to get started is to download and install
VirtualBox and vagrant.  Both are available as dmg files and you can get up and
running pretty quickly this way.

I highly recommend using the Vagrant documentation for "Getting Started" to move
through the initial setup.  It's straight forward and doesn't bear repeating here.
For my project I'm using Ubuntu precise 64-bit.  You will need to add the box if
you don't have it already:

{% highlight bash %}
$ vagrant box add precise64 http://files.vagrantup.com/precise64.box
{% endhighlight %}

Then, in my <code>VagrantFile</code> I added the following code:

{% highlight ruby %}
Vagrant.configure("2") do |config|
  config.vm.box = "precise64"
end
{% endhighlight %}

Try it out by starting up the box and ssh'ing into it:

{% highlight bash %}
$ vagrant up
$ vagrant ssh
{% endhighlight %}

Now you probably want to do a little more than start up an empty box.  For me
I want to run a boostrap script to install necessary packages and configure
the machine and I also want to forward some ports so I can access my dev site
from my browser.  Here is the full VagrantFile I have set up:

{% highlight ruby %}
Vagrant.configure("2") do |config|
  config.vm.box = "precise64"
  config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.network :forwarded_port, guest: 8000, host: 8001
  config.vm.provision :shell, :path => "deploy/bootstrap.sh"
end
{% endhighlight %}

You can see I have my bootstrap script at <code>deploy/bootstrap.sh</code>.
I keep my deployment files in a <code>deploy</code> folder at the same
level as my VagrantFile.  This is mainly for tidiness.  This is also where
I keep my <code>deploy/requirements.txt</code> file that maintains the
package list for my python deployment.  Now a short
bootstrap example which you'll find easy to modify and get started with.

{% highlight bash %}
#!/bin/bash
                               
set -e

echo
echo "Environment installation is beginning.  This may take a few minutes ..."

##
#   Install core components    
##

echo
echo "Updating package repositories ..."
apt-get update

echo
echo "Installing required packages ..."
apt-get install -y apache2 git make postgresql

##
#   Configure postgresql
##

echo
echo "Configuring postgres ..."
sudo -u postgres psql -e -c "CREATE USER vagrant WITH PASSWORD 'vagrant';"
sudo -u postgres psql -e -c "CREATE DATABASE vagrant ENCODING 'UTF-8';"
sudo -u postgres psql -e -c "GRANT ALL PRIVILEGES ON DATABASE vagrant TO vagrant;"

##
#   Install python packages
##

echo
echo "Installing ubuntu python packages ..."
apt-get install -y python \
                   python-dev \
                   python-pip \
                   python-software-properties

echo
echo "Installing python packages ..."
mkdir -p /home/vagrant/.pip_download_cache
export PIP_DOWNLOAD_CACHE=/home/vagrant/.pip_download_cache
export VIRTUALENV=/home/vagrant/env
pip install -U pip virtualenv
virtualenv --system-site-packages $VIRTUALENV
source $VIRTUALENV/bin/activate
pip install -r /vagrant/deploy/requirements.txt
if [ $? -gt 0 ]; then
    echo 2> 'Unable to install python requirements from requirements.txt'
    exit 1
fi

##
#   Setup is complete.
##

echo
echo "The environment has been installed."
echo
echo "You can start the machine by running: vagrant up"
echo "You can ssh to the machine by running: vagrant ssh"
echo "You can stop the machine by running: vagrant halt"
echo "You can delete the machine by running: vagrant destroy"
echo
echo "If this is your first time, you should install the virtual machine guest additions."
echo "To do that, ssh into the machine (vagrant ssh) and run: sudo ./postinstall.sh"
echo
exit 0
{% endhighlight %}

It's important to understand that the <code>/vagrant</code> directory in your
Ubuntu installation is the same directory where your VagrantFile can be found.
This allows you to use your favorite editor, say [SublimeText2][sublimetext],
on your mac to edit the files being served in your Ubuntu environment.

Now go forth and start developing!

[ubuntu]: http://www.ubuntu.com/
[virtualbox]: https://www.virtualbox.org/wiki/Downloads
[vagrant]: http://downloads.vagrantup.com/
[sublimetext]: http://www.sublimetext.com/
