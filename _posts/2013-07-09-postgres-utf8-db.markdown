---
layout: post
title:  "Creating a Postgres DB with UTF8 Encoding"
date:   2013-07-09 20:30:00
categories: postgres utf8
comments: true
---

Today I had to figure out how to [create a db in postgres][postgres] with UTF8
encoding.  My first attempt was to do this:

{% highlight bash %}
/usr/bin/sudo -u postgres /usr/bin/psql -e -c "CREATE USER vagrant WITH PASSWORD 'vagrant';"
/usr/bin/sudo -u postgres /usr/bin/psql -e -c "CREATE DATABASE vagrant OWNER vagrant ENCODING 'UTF-8';"
{% endhighlight %}

Of course I got this arcane message:

{% highlight bash %}
CREATE DATABASE vagrant OWNER vagrant ENCODING 'UTF-8';
ERROR:  encoding UTF8 does not match locale en_US
DETAIL:  The chosen LC_CTYPE setting requires encoding LATIN1.
{% endhighlight %}

I wasn't quite sure what to do with this but some helpful googling got me here:

{% highlight bash %}
/usr/bin/sudo -u postgres /usr/bin/psql -e -c "CREATE USER vagrant WITH PASSWORD 'vagrant';"
/usr/bin/sudo -u postgres /usr/bin/psql -e -c "CREATE DATABASE vagrant OWNER vagrant ENCODING 'UTF-8' LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8' template template0;"
{% endhighlight %}

This worked perfectly for me.  Hope this helps!


[postgres]: http://www.postgresql.org/docs/9.2/static/sql-createdatabase.html
