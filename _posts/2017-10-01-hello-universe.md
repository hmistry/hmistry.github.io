---
layout: post
title:  "Hello Universe!"
date:   2017-10-01
categories: misc
tags: misc
excerpt_separator: <!--more-->
---

{% highlight ruby %}
  puts "Hello Universe!"
{% endhighlight %}

Stay tuned... 😃

Testing the timeago plugin on GitHub Pages

{% assign date = '2020-04-13T10:20:00Z' %}

- Original date - {{ date }}
- With timeago filter - {{ date | timeago }}
