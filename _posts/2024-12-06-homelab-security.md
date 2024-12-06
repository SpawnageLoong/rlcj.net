---
layout: post
date: 2024-12-06 12:00:00 +0800
title: 'Setting up a Homelab - Traefik and PiHole'
tags:
  - homelab
hero: /uploads/2024-12-06-homelab2/hero.jpg
overlay: blue
---

Remember how I said I was using Seafile? I changed my mind. I slept on the idea and realised I'm going to be putting in more effort to set it up than if I were to use Nextcloud. With how popular nextcloud is, there's way more community support and documentation available, so I decided to switch over to Nextcloud.

Removing Seacloud was as easy as removing the containers in Docker, and deleting the seacloud folders in `/etc/`.

But before I go on to setting up Nextcloud, I want to set up the network to allow for remote access. And at the same time, I'll implement the security measures. This took quite a lot of googling since I'm going in pretty much completely blind, but thankfully there are guides out there that are relatively easy to follow.

## The googling

Before I could actually do anything, I had to decide what I was going to do. A google search for "how to access self hosted remotely" whould give me more than a few different ways, and a lot of these "methods" were different software doing similar things. The two methods that stood out the most to me were using a VPN, and using a DDNS/reverse proxy/idk what else.

The VPN method was something I've tried before, and it wasn't something I'd like to try again. Setting up wireguard as the VPN was tricky for me and I just couldn't get it working the last time. I also didn't like the idea of having to connect to a VPN every time I wanted to access my server. Connecting to a VPN might sound like a minor inconvenience, but minor things add up.

The DDNS method on the other hand was extrememly unintuitive for complete newbies like me. I went down a deep rabbit hole of googling what all of these terms and software names were, and things got even more complicated when I realised this was like deciding on a tech stack for a web app. There are so many different software doing similar things, and it's "up to you" to create your own stack. Like hell a noob like me can decide anything.

And to make matters worse, a lot of guides are how-tos without explaining what you're actually doing or why you're doing it. I don't want to blindly follow a guide and not understand what I'm doing. If anything goies wrong after an update or a typo, I won't be able to fix it if I don't understand what I'm doing. Not to mention the security risks of blindly executing `sudo` commands on security features like your firewall.

## Enter Cloudflare Tunnels

After taking a step back from googling, I thought to narrow down my options by adding more criteria to my searches. I was already a Cloudflare customer, so I figured I'd see if there were specfific guides for Cloudflare users. And lo and behold, I stumbled upon Cloudflare Tunnels.

Cloudflare Tunnels is a service (that's available for free) that allows you to connect specific services on your server to Cloudflare's network. This means that you can access these services from anywhere in the world, as long as you have an internet connection. It's like a VPN, but you don't have to connect to it manually. You run a service on your server called `cloudflared` that connects to Cloudflare's network, and Cloudflare routes the traffic throught that tunnel to your server.

This sounded like a much easier way to accomplish what I wanted, so I gave it a shot. Setting up the tunnel was pretty easy using the [official docs][cf-tunnel-docs] and I had a tunnel on a subdomain that I owned. I tried routing the tunnel to my server's portainer instance and whoops, it doesn't work.

![tunnel error](/uploads/2024-12-06-homelab2/cf-tunnel-502.png)

At least it's giving me a Cloudflare error page, so I know that cloudfare is working. Looks like the issue is on my end, so I'll have to figure out what's wrong. A quick check of the docker logs for cloudflared told me that it was receiving the connection request, but there was likely a routing issue. I tried various service addresses to see what they would do, like `localhost:9443`, `[IP-ADDRESS-HERE]:9443`(not revealing that), `homelab.local:9443`, and they all gave different errors. 

A few more google searches and I figured it was likely because there's no DNS resolution for external connections. But at the same time I have no way of actually knowing if that's the issue. So I put the tunneling on pause and went back to googling.

## Googling 2: Electric Boogaloo

I went back to googling about setting up remote access via cloudflare and found a Reddit post that claimed that Traefik with Cloudflare Tunnels was "all the rage" at the time. I'd heard of Traefik before on a homelab tour video on Youtube, so I figured it's worth a google search. Another few Google searches later, and I found TechnoTim's [video guide][technotim-traefik-video] on setting up Traefik 3 with Cloudflare. The thing I like about TechnoTim's videos is that he explains what he's doing and why he's doing it, so I can follow along and understand what's going on, and his website has written materials for me to copy-paste easily.

Before we get to that though, I just want to highlight a [reply][reddit-reply] I found on Reddit that sums up my feelings on self hosting really well, especially the difficulty:

> But there are a lot of people here who, when reading a question asked by a real amateur fling out a "oh you just need to create a thingummy and verify your watchamacallit then connect it to your hootenanny but don't forget to close off your ballyhoo or you'll lose access to your fadoodle" while totally forgetting that each of those steps has its own huge rabbit hole of understanding and implementation and that actually acts as a barrier to someone getting going rather than helping them. Not saying those answers are bad or wrong btw, just that they can be totally overwhelming for a noob. - CrispyBegs, 2023

Anyway, back to TechnoTim.

The setup he was doing didn't actually involve Cloudflare Tunnels, or even remote access. But it did involve setting up Traefik to route traffic to different services on his server. This sounded relevant to my previous issue, so I went ahead and followed along with his guide. It was really straight forward right up until he started using PiHole as a DNS server. I don't have PiHole set up yet, and he didn't explain how t oset it up, so I paused the video there and went to look into setting up PiHole.

## PiHole (and a rant)

I've used PiHole before on a Raspberry Pi, so I'm familiar with what it does. But it previously was a standalone device (that's not used anymore because home-level adblocking doesn't work if the parents _want_ the ads), and now I was setting it up as a piece of software on a server.

Before trying to install PiHole, I wanted to try and install AdGuard Home. This is a similar software that I had used back when I was living in Bangkok, and it was (supposedly) more feature rich, so might as well right? And it even has a Docker image ready for me to use!

Again, reality come in and slaps me in the face. The docker run command they provide has more than a few ports that had to be mapped, and more than one of those ports were already in use. Port 53 was the first one, and that was a known issue with Ubuntu since there was `system-resolved` that ran by default, so you were supposed to change to config for it to not use port 53. I did that, and then I ran into the issue of ports 67 and 80 being in use. Another rabbit hole of googling later, and I got adguard home running on my server. I went through the initial setup on the web UI, and BAM! It's not working anymore.

After that inital setup, I could no longer access the web UI via port 8080 (I had to map 8080 to the web UI to avoid conflicts). It would just throw me a 404 error. That was it, I give up. Back to PiHole.

And lo and behold, THE EXACT SAME ISSUES WITH PORT CONFLICTS. Thankfully, resolving these conflicts was easier since I'd done the same thing with AdGuard Home. I got PiHole up and running, aaaaaaaand it's not working. After a quick search for why I can't access the web UI, I realised it's not `homelab.local:8080`, but instead you have to use `homelab.local:8080/admin`. I don't know why they don't just redirect you to the correct page, but whatever.

Anyway I tested PiHole by changing my DNS settings on my desktop to point to the server, and it worked. I could see the queries coming in and being blocked, so I knew it was working. Now I could go back to TechnoTim's video.

I went ahead and setup the DNS routes in PiHole, aaaaand it's not working. I can't connect to traefik at its configured address. I tried a few different things, like changing the port, changing the address, but nothing worked. I even tried to access the traefik dashboard directly, but that didn't work either. I'm at a loss here, so I'm going to have to go back to the drawing board. Then I saw the clients list in PiHole. 

There were no clients except my desktop and the pihole container itself. Then I remembered that Docker containers handle networking differently. I threw my hands up in surrender and gave up on containerised PiHole. Back to bare metal.

I removed the PiHole containers and went ahead and installed PiHole using their automated installer. Again, I had to change port 80 to 8080 to avoid conflicts and boom. It's working. I can access the web UI, I can see the queries coming in, and I can see the clients. I set up the DNS routes in PiHole, and now I have access to the traefik dashboard. I can't seem to get nslookup to work with the local DNS server, but I can still access the web UI so I'm going to ignore that issue for now.

## Finally setting up Traefik

Now that I can access traefik, I can now follow along with TechnoTim's video. I changed the certificated from staging to production and all works as expected.

Now to test it with a new service. I'm still following the video and setting up an nginx service using the nginx-hello image. I do have to deviate from the cideo slightly as Tim uses port 8080 for this nginx service, but I'm using port 8081 as 8080 is being used by PiHole. 

{% highlight yaml %}
services:
  nginx:
    image: nginxdemos/nginx-hello
    ports:
      - 8081:80
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nginx.rule=Host(`nginx-hello.local.rlcj.net`)"
      - "traefik.http.routers.nginx.entrypoints=https"
      - "traefik.http.routers.nginx.tls=true"
      - "traefik.http.services.nginx.loadbalancer.server.port=8081"
    networks:
      - proxy

networks:
  proxy:
    external: true
{% endhighlight %}

Again, setup the local DNS records in PiHole and again, nslookup isn't working. But now the nginx also isn't working. It returns a 502 bad gateway. Both the traefik dashboard and pihole query logs show that everything is fine, so it's likely an issue with the nginx container. I removed my changes to the docker compose file and tried again, and it worked. Turns out the issue was with the port mapping. The containers are running on the docker network called `proxy`, so I don't have to worry about port conflicts with PiHole which is running bare metal. I'll probably run into port conflicts later, but that's future me's problem.

Back to the video, Tim goes on with using Traefik for workloads outside of docker, but I'm skipping that for now since I plan on keeping everything in docker. I'll probably come back to this later, but for now I'm happy with what I have.

## Back to Cloudflare Tunnels

Now, let's try setting up Cloudflare Tunnels again. I went back to the cloudflare dashboard, updated the tunnel to point to the nginx service, and it still doesn't work. I opened up the logs for cloudflared, and I noticed these errors:

`ERR  error="Unable to reach the origin service. The service may be down or it may not be responding to traffic from cloudflared: dial tcp: lookup nginx-hello.local.rlcj.net on 192.168.1.1:53: no such host" connIndex=2 event=1 ingressRule=1 originService=https://nginx-hello.local.rlcj.net`

It seems to be using the DNS server at `192.168.1.1` which is my home router, rather than my PiHole. I tried to fix this by changing the DNS settings in `/etc/systemd/resolved.conf` to point to the PiHole, but that didn't work. 

Using `cat /etc/resolv.conf` showed that the home router was still showing up as a nameserver. After fiddling around a lot, I found a [post][cf-post] in the cloudflare community that said Docker uses the same DNS as the host machine, but my docker instance hadn't been restart since I changed the DNS settings. I restarted my whole homelab machine and it still doesn't work. But at least the logs show that it's using `8.8.8.8` as the DNS server instead of the home router, so at least something has changed.

Except I never used `8.8.8.8` as any of my fallback DNS servers and I don't know where it's getting that from. `8.8.8.8` doesn't show up in `/etc/resolv.conf`, so I'm at a loss here.

I decided to pause here because it feels like I'm going nowhere. I have an idea to move all of my existing services to use traefik, then put cloudflred inside the proxy network and see if that works. But that's a problem for another day.

<hr>

[cf-tunnel-docs]: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/
[technotim-traefik-video]: https://www.youtube.com/watch?v=Vc9m6g6YpFc
[reddit-reply]: https://www.reddit.com/r/selfhosted/comments/133rr6n/comment/jieshxj/
[cf-post]: https://community.cloudflare.com/t/cloudflare-tunnel-not-using-local-dns-for-resolution/600141
