---
layout: post
date: 2024-11-19 13:29:00 +0800
title: 'rlcj template for Jekyll'
tags:
  - project
  - rlcj-template
hero: /uploads/generic-hero-images/akihabara.jpg
overlay: purple
---

A modern, fast and configurable Jekyll theme for personal blogs.

{: .lead}

This is placeholder text for now. I will update this post with more information about the project soon.

Project posts:
<ul>
  {% for post in site.tags.rlcj-template %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span>{{ post.date | date: "%d %b %Y" }}</span>
      
    </li>
  {% endfor %}
</ul>
