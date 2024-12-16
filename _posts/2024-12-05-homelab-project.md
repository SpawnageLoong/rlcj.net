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

A home server being set up to host various services, including a website, file sharing, and more.

{: .lead}

Services include:

- PiHole for network-wide ad blocking and local DNS
- Traefik for reverse proxy
- Portainer for managing Docker containers
- Homepage for dashboards, both internal and external
- Nextcloud for file sharing and syncing
- Vaultwarden for password management
- Jellyfin for media streaming

Project posts:
<ul>
  {% for post in site.tags.homelab %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <span>{{ post.date | date: "%d %b %Y" }}</span>
      
    </li>
  {% endfor %}
</ul>