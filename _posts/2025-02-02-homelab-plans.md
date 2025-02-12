---
layout: post
date: 2025-02-02 12:00:00 +0800
title: 'Making plans for my homelab'
tags:
  - homelab
hero: /uploads/2025-02-02-homelab-plans/hero.png
overlay: blue
---

It's been a while since my last post. I've been working on a few things, like an RC Car project, but that's not what I'm writing about today. I might get around to writing about that project in the future, but today I want to talk about my homelab.

I haven't needed to touch my homelab since my last post, and it's been running smoothly since then. I've been able to completely ignore it and it just works. But then I saw [this video](jeff-geerling-rack-video) by Jeff Geerling, and my brain had just found it's new hyperfixation.

I have absolutely no need for a rack in my homelab. I'm only running a single server hooked up to a wireless AP, and that's all I need. But I want a rack. I want to have a rack loaded with patch panels and networking equipment. I want to cosplay as a sys admin. So I immediately looked at getting myself the rack that Jeff had in his video, and immediately vomitted blood at the price. 

![Shipping costs](/uploads/2025-02-02-homelab-plans/deskpi-shipping.png)

US$120 for a rack was a bit pricey for me, but it was still "reasonable enough". But  
US$104 for shipping? No way. I'm not forking out US$230 for something like this. 

I tried checking other retailers, but the only other place I could find it was Amazon US, and the base price was even higher. I then tried looking at other racks and found Novaspirit Tech's [video](novaspirit-rack-video) on his 10-inch rack. Unfortunately, the rack that he used was also only available on Amazon US, and it would not ship to Singapore. At this point, I decided to put the rack idea on hold, and instead focus on what I would be populating the rack with.

## The UniFi idea

My first thought was to replace my current router/AP with a more prosumer grade model. Right now I'm using a Huawei WiFi AX3, which does its job perfectly fine, but lacks features like VLANs that I want to play around with. I looked at Ubiquiti's lineup, and decided on the UniFi Cloud Gateway Ultra (UCG-Ultra). It's a bit overkill for my needs, but it was the second cheapest UniFi gateway available here. The cheapest one was the UniFi Express, which comes with a built-in AP, but reviews online seem to suggest that it has overheating issues and generally performs worse than the UCG-Ultra, even though they should have the same processor.

I decided to go with UniFi's lineup because it seemed to be the most represented in the YouTube videos I've watched, and the UI looked pretty good.

Since the UCG-Ultra didn't have a built-in AP, I would have to get a separate AP. Again, I decided to go with UniFi, and chose the UniFi U6+ since it was the cheapest UniFi AP with full WiFi 6 capabilities. There were cheaper APs, like the Swiss Army Knife, but those models didn't have the 2.4Gbps data rate that WiFi 6 is capable of. Admittedly, the U6+ only has a single GbE port, so I'd never be able to fully utilize the 2.4Gbps data rate, but it's still nice to have.

Now that I'd decided on an AP, I had to get a switch to connect everything together. I was still looking at UniFi's lineup, but had a much harder time deciding on a switch. I wanted it to have both 2.5GbE and PoE capabilities, but the only switch that had both was the Enterprise 8 PoE, which was way out of my budget (I didn't look at the rack mounted models because I know they would be waaaay out of budget). 

## Looking for cheaper alternatives

I realised at this point that I was going to be throwing a lot of money at this project, and I wasn't sure if I wanted to do that. So I did a 180-degree turn and went to look at TaoBao. The network switches were way waaaay cheaper there, and I could get an 8-port 2.5GbE managed switch with PoE for less than SG$100. That led me down the rabbithole of looking at a lot of other equipment, like more mini-pcs.

It was now nearing the end of January and Chinese New Year was coming up soon. If you didn't know, China pretty much shuts down for two weeks during Chinese New Year, and if I wanted to get anything from TaoBao, I had to order it soon. I didn't see a point in getting the switch now, since I didn't have any other equipment to connect to it, so I instead decided to get another N100 mini-pc. I found one that had five 2.5GbE ports, and decided to get that. It came in a full metal enclosure that doubled as a heatsink and could be passively cooled, which was a huge plus for me. Now all I had to do was wait for it to arrive.

![N100 mini-pc](/uploads/2025-02-02-homelab-plans/taobao-n100.png)

Luckily for me, it got out of China just before the Chinese New Year shutdown, and I got it on the third day of Chinese New Year. At this point I realised I didn't actually have a plan for it. I installed OPNsense on it for lack of a better idea, and while it worked fine, I didn't really have a use for it. I couldn't even use it as a router because I have a primary router provided by my ISP that I'm not willing to replace. I only want to tinker with the network devices in my room, and I don't want to affect the rest of the house. So now it's just sitting on a desk gathering dust.

## The current state of my homelab

I realised at this point that I didn't actually have a plan for my homelab. I was just buying things that I thought were cool, and not actually using them. I figured now would be a good time to summarise the current state of my homelab, and make a plan for its future.

### The physical setup

![Physical setup](/uploads/2025-02-02-homelab-plans/Homelab-physical.png)

The center of my current homelab is a Huawei router in bridge mode. It's connected to my home's main router through a Gigabit network switch. I have an N100 mini-pc that runs all of my services. The 2.5GbE port on the N100 mini-pc is connected to a 1GbE port on the bridge mode router. My gaming PC (which has a 2.5GbE port) is also connected to a 1GbE port on the bridge mode router. 

### The software setup

![Network diagram](/uploads/2025-02-02-homelab-plans/Homelab-network.png)

All of the traffic from outside the network goes through Cloudflare, and enters the N100 mini-pc though a Cloudflare tunnel using the `cloudflared` docker container. The traffic is routed through Traefik, which acts as a reverse proxy, sending the traffic to the appropriate docker container. The N100 mini-pc also runs Pi-hole, which blocks ads on the network and handles DNS for my local devices.

## Future plans

![Future plans](/uploads/2025-02-02-homelab-plans/Homelab-hardware-plan.png)

This is the long term goal for my homelab's hardware. First, I want to replace the Huawei router with a standalone network switch and standalone AP. I've already ordered an 8-port 2.5GbE managed switch with PoE, but it's still CNY so it'll take a while to arrive. I'm still looking for a standalone AP, but I'm not in a hurry to get one. I can stick with the Huawei router just for the WiFi for now.

After the switch is set up, I'll be installing Proxmox on the N100 mini-pc. I'll likely move my current docker setup to Proxmox, and run all of my services in LXC containers. I'll also be setting up a VPN server on one of my servers so I can access my home network from outside.

After that, I'll be setting up a rack to house all of my equipment. I saw a [video](diy-rack-video) on making a rack using 3D printed parts, and I think I'll be going with that. I've ordered two pairs of 9U rack rails, and I'll be printing the rest of the parts. I use a Bambulab X1C printer, and it'll be able to handle the parts just fine.

After that, I'll be looking at making a custom NAS. I've seen a few reasonably priced ITX motherboards for NAS builds, and I think I'll be going with that. I'll be buying some 5.25" hard drive cages that are meant for optical drive bays, and I'll be using those to hold my hard drives. I plan to get 4TB Seagate Exos drives since they're pretty cheap and should be reliable given that they're meant for datacenters. It'll be running an Unraid VM in Proxmox, and the drives will be in a RAID 5 configuration. I intend for all of the containers on Proxmox to be able to access the NAS, so I'll likely be setting up a Samba share on the NAS. This is a long term goal, and I don't expect to get to it anytime soon, so details are still up in the air.

And finally, I'll get a fourth mini-pc to run Proxmox Backup Server. By then, I should have three mini-pcs running Proxmox, and I'll be able to set up a cluster, so having a backup server would be a good idea. I don't have any plans for this mini-pc yet, so I might end up using a Raspberry Pi instead.

I don't have any plans to get a UPS right now, since I don't plan on running anything mission critical on my homelab. If my house loses power, the internet will go down anyway, so there's no point in having a UPS. I might change my mind in the future, but for now, I don't see a need for one.

I'll be writing more about my homelab as I make progress on it, so stay tuned for more updates.

<hr>

<a href="https://wphostingreviews.com/wp-content/uploads/2014/05/blog-hosted-or-selfhosted.png">Hero image</a> taken from <a href="https://wphostingreviews.com/guides/whats-best-for-you-hosted-or-self-hosted-wordpress-hosting/">WP Hosting Reviews</a>.

[jeff-geerling-rack-video]: https://www.youtube.com/watch?v=y1GCIwLm3is
[novaspirit-homelab-video]: https://www.youtube.com/watch?v=utOraP1T9sA