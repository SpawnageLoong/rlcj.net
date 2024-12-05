---
layout: post
date: 2024-12-05 12:00:00 +0800
title: 'Homelab'
tags:
  - project
  - homelab
hero: /uploads/2024-12-05-homelab1/homelab-hero.jpg
overlay: green
---

A home server being set up to host varios services, including a website, file sharing, and more.

{: .lead}

This is placeholder text for now. The homelab is still being set up and I'm still figuring out what services I want to run on it. I'll be documenting the process here, so stay tuned!

Project posts:
<ul>
  {% for post in site.tags.homelab %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span>{{ post.date | date: "%d %b %Y" }}</span>
      
    </li>
  {% endfor %}
</ul>