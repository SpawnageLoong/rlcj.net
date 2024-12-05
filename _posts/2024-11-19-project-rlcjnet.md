---
layout: post
date: 2024-11-19 14:45:00 +0800
title: 'rlcj.net'
tags:
  - project
  - rlcj
hero: /uploads/generic-hero-images/nordic-lake.jpg
overlay: orange
---

The website you're reading right now!

{: .lead}

This is placeholder text for now. I will update this post with more information about the project soon.

Project posts:
<ul>
  {% for post in site.tags.rlcj %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span>{{ post.date | date: "%d %b %Y" }}</span>
      
    </li>
  {% endfor %}
</ul>
