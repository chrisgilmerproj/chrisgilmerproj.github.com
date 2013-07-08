---
layout: post
title:  "Deploying Django with Nginx + uWSGI"
date:   2013-07-07 20:30:00
categories: django nginx uwsgi
comments: true
---

It's pretty common that when I want to deploy a [Django][django] project
that I'll want to use the [Nginx][nginx] + [uWSGI][uwsgi] combo.  For reference
I'm going to include a few rules for setting this up on an Ubuntu system.

For this to work you should know two things.  The first is that I expect your
django project to reside in <code>/var/www/project</code> and that your
project's virtual environment is at <code>/var/www/project/env</code>.  I also
expect that you've installed python and you can run your django project
with the standard <code>python manage.py runserver 0.0.0.0:8000</code> command.
This tutorial will replace that command.

To get started you want to install Nginx, uWSGI, and the uWSGI python plugin.

{% highlight bash %}
sudo apt-get update
sudo apt-get install -y nginx uwsgi uwsgi-plugin-python
{% endhighlight %}

After installing you'll want to write an Nginx configuration file for your
django project.  In this case the upstream should point to the unix
socket that you want to use to serve your django project.

{% highlight nginx %}
# upstream django server
upstream django {
    server unix:///run/uwsgi/app/uwsgi/socket;
    }

# configuration of the server
server {
    # the domain name it will serve for
    server_name localhost;
    listen      8000;
    charset     utf-8;

    # max upload size
    client_max_body_size 15M;

    # Django static
    location /static {
        alias /www/var/project/static;
        expires off;

        # fix issue with Vagrant and sendfile
        sendfile off;
    }

    # Django app
    location / {
        uwsgi_pass  django;
        include     /etc/nginx/uwsgi_params;
        }
    }
{% endhighlight %}

Link this file and start Nginx:

{% highlight bash %}
ln -s /www/var/project/deploy/nginx.conf /etc/nginx/sites-enabled/
sudo /etc/init.d/nginx start
{% endhighlight %}

Next you need to make a python uWSGI file.  Place this file at
<code>/var/www/project/wsgi.py</code>.  It should be at the same level
as your settings.py file.  In Django 1.5 and later this file is generated
for you when you start a new django project.

{% highlight python %}
import os
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "settings")
from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
{% endhighlight %}

Then you want to write yourself a uWSGI ini file.  The unix socket is
determined by default by the uwsgi process and resides at 
<code>/run/uwsgi/app/uwsgi/socket</code>.

{% highlight ini %}
[uwsgi]

##
# Django-related settings
##

# the base directory (full path)
chdir           = /var/www/project
# Django's wsgi file
module          = wsgi
# the virtualenv (full path)
home            = /var/www/project/env

##
# process-related settings
##

# master
master          = true
# maximum number of worker processes
processes       = 10
# clear environment on exit
vacuum          = true
# automatically kill workers on master's death
no-orphans      = true
# set mode of created UNIX socket
chmod-socket    = 660
# place timestamps into log
log-date        = true
# user identifier of uWSGI processes
uid             = www-data
# group identifier of uWSGI processes
gid             = www-data
# daemonize and log output
daemonize       = /var/log/uwsgi/project.log
{% endhighlight %}

Then link the uWSGI file and start uWSGI.

{% highlight bash %}
ln -s /var/www/deploy/uwsgi.ini /etc/uwsgi/apps-enabled/
sudo /etc/init.d/uwsgi start
{% endhighlight %}

Now you should be able to browse to http://0.0.0.0:8000 and see your
django project!  If you make changes and want to reload you can simply run
<code>sudo /etc/init.d/uwsgi reload</code>.

One thing that I noticed while deploying was that my static files did not serve
completely, meaning that my javascript files seemed to be cut off.  After doing
a little bit of searching it appears that a bug with Vagrant, nginx, and shared
folders means you need to add <code>sendfile off;</code> to the static file
section of your nginx.conf.  I initially thought this was a problem with
<code>grunt</code>, which I use for generating static files, but after checking
there was simply no problem with the generated files.

[django]: https://docs.djangoproject.com
[nginx]: http://nginx.org/en/docs/
[uwsgi]: https://uwsgi-docs.readthedocs.org/en/latest/
