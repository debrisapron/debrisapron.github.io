---
layout: default
title: DEBRIS APRON
---

Hi, I'm Matthew aka DEBRIS APRON aka Simon Break. I'm a freelance JavaScript / Ruby / Rails / etc developer living between London and Los Angeles. I play music as [THE EUROPEAN](http://iamtheeuropean.com). I like programming, cooking, nature and pretending to be an elf with a bunch of grown adults. You could probably hire me to do some programming or whatever.

Here is a blog post that I wrote:

<ul class="post-list">
  {% for post in site.posts %}
    <li>
      <a class="post-link" href="{{ post.url }}">{{ post.title }}</a>
      <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>

If I ever write another one you can be notified if you subscribe <a href="/feed.xml">via RSS</a>.