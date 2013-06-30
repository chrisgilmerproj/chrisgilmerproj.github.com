---
layout: post
title:  "Migrating to Jekyll"
date:   2013-06-28 21:25:03
categories: jekyll update
---

I've decided to migrate my project blog to use [Jekyll][jekyll] and deploy with 
[Github Pages][gh-pages].  There are a lot of reasons for this including 
ease of use, version control, and absolutely no reliance on using a sql
database. Primarily I'm making this move to make it easier to post about what I
do at work and at home relating to coding, making and related projects.  It's
been nearly 3 years since I've managed a blog and it's time I returned to it.

The [Jekyll docs][jekyll] are easy to read and installing was a
snap.  With a few modifications I was easily able to set up a blog that worked
for me and migrate all my old wordpress posts.

One thing I really wanted to do was to show my latest post on the homepage
of my blog.  I found this to be a little easier than expected.  While there
are no convenience methods for grabbing the latest post you can always
limit the number of posts you grab in a query.  To do so I edited my 
<code>index.html</code> file to look like this:
 
{% highlight html %}
{% raw %}
<div id="home">
    {% for post in site.posts limit:1 %}
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
      <p class="meta">{{ post.date | date_to_string }}</p>
      <div class="post">{{post.content}}</div>
    {% endfor %}
  <h3>Latest Posts</h3>
  <ul class="posts">
    {% for post in site.posts limit:5 %}
      <li><span>{{ post.date | date: "%Y-%m-%d" }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
      <li><span>earlier</span> &raquo; <a href="/archive.html">Archive of Posts</a></li>
  </ul>
</div>
{% endraw %}
{% endhighlight %}

I'm also looking forward to using the built-in syntax highlighting that
leverages the Pygments library.  I likely will provide examples in bash
and python as well as config files in my code but here's the ruby example
that Jekyll provides for you.

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Chris')
#=> prints 'Hi, Chris' to STDOUT.
{% endhighlight %}

Check back here for more posts.  

[jekyll]:    http://jekyllrb.com
[gh-pages]:  http://pages.github.com/
