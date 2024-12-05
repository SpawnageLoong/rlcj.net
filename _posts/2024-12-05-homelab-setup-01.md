---
layout: post
date: 2024-12-05 15:45:00 +0800
title: 'Setting up a Homelab - Initial setup'
tags:
  - homelab
hero: /uploads/2024-12-05-homelab1/hero.jpg
overlay: purple
---

I was in the middle of reorganising my bedroom-workshop when I found my unused mini-PC. I originally got it for use as an all-in-one homelab, with proxmox for virtualisation, a Pfsense router, and a NAS, but I never managed to get it working. It was a pain to manage and was way more effort than it was worth, so the mini-PC was left to gather dust.

At this point of time I still don't really have a compelling use for it because I just pay for externally hosted services or I get lots of student benefits for free. But I'm graduating soon and I'll lose access to those benefits, so I might as well start working on something now and if it works out, cool - I can save some money by self-hosting. If it doesn't, I can just shut it down and not worry about it.

Do note that this isn't a guide, but more of a documentation of my journey. I'm not an expert in this field, so I'll probably make mistakes along the way. I'll try to document them as well, so that others can learn from them. There is definitely not enough info in here to act as a guide for those who are new to this, but I'll try to provide links to the resources I used.

## The Mini-PC

The hardware I'm working with is a Chuwi Larkbox X with an Intel N100. It's a pretty small box and conveniently fits in a gap at the end of my row of drawers. It has 4 cores and 4 threads, 12GB of RAM, and 1TB of storage. I have no clue what I'm going to use it for right now, but these specs should give me room to expand.

## The Plan

While I don't have a concrete list of services I intend to run, I do have a rough list of criteria that I want to meet:

1. **Low Maintenance**: I don't want to spend time managing this server. I want to set it up and forget about it.
2. **Remote Access**: I want to have certain services accessible from outside my local network, like a file sharing service or a self-hosted website (hint hint).
3. **Secure**: I want to make sure that the server is secure and that I'm not exposing myself to any unnecessary risks. I'll also use this to learn more about network security.
4. **Useful**: I want to run services that I'll actually use. I don't want to run something just because I can.

## Ubuntu Server

To start off, I installed Ubuntu Server 24.04.1 LTS on the server (from now on, I'll refer to the mini-PC as the server). I chose Ubuntu because it's what I'm most familiar with and it's safe to assume that most software will be able to run on it. The installer was pretty straightforward and I didn't run into any issues. It also provides the option to install OpenSSH during the installation process, which is a nice touch. At the same time, I also enabled Ubuntu Pro for the server. I honestly don't know exactly what it does beyond live patching and extended security support, but I have a free license for it so I might as well use it. Also worth noting, I already had a static IP set up for the server, so I didn't have to worry about that.

I went ahead and tested the SSH connection from my desktop using both the static IP and the hostname (I used `homelab.local`) and it worked perfectly, so I could disconnect the display cable and keyboard from the server, leaving just the network cable and power cable connected.

## Docker

After the usual `sudo apt update` and `sudo apt upgrade`, I installed Docker Engine using their apt repository following the [documentation][docker-engine-docs]. I've used Docker before, and the main thing I like about it is being able to use Portainer. Portainer makes managing containers way easier, and with so many services available as containers, this makes long-term management a lot easier. I installed Portainer as a container following the [documentation][portainer-install-docs], and I was able to access it from my desktop using both the static IP and hostname.

## Homepage

Next, I installed [homepage][homepage-link] by gethomepage. It's a self-hosted dashboard that's used to display server information and an app launcher for other services on the server. This is also really easy to do with Docker. I just followed the documentation, but used the docker run command instead of docker compose because I'm not familiar with docker compose. It's not explicitly stated in the documentation, but you should create a config folder somewhere on your server for each container. For me, I just used `~/container-data/CONTAINER_NAME` for each container.

I was able to access homepage from my desktop via the hostname, at which point I noticed two things. First, the homepage container was running in the terminal, as in it wasn't running in the background and would exit once I did a Ctrl+C. Second, homepage was telling me I only had 100GB of storage, which is definitely not what my drive should have.

The first issue is easy enough to fix since I have portainer running. I exit the container in the terminal, and restart it in portainer's web UI. At the same time, changing its restart policy to "Unless Stopped", so it will automaically start when the server boots up.

The second issue was a bit less obvious. First, I checked the storage using `df -h --total`, and it agreed with homepage, only showing 97GB of storage available. A quick google search later, and I found a reddit post saying that this is the default behaviour of Ubuntu Server installs. It only allocated 100GB during the install, and the rest of the storage is left unallocated. Another google search later on how to extend the LVM partition, and I had a [guide][lvm-extend] to follow now.

I followed along with the guide and now I've got the full 1TB of storage available.

![default homepage](/uploads/2024-12-05-homelab1/homepage.png)

Now that it's all up and running, I have to change the homepage to actually show useful information and useable shortcuts. This is the `services.yaml` file I'm using:

{% highlight yaml %}
- Server Management:
    - Portainer:
        href: https://homelab.local:9443/
        description: Manage Docker containers.
        container: portainer
        icon: portainer.png
        showStats: true
    - Homepage:
        href: homelab.local:3000/
        description: The page you're on right now.
        container: homepage
        icon: homepage.png
        showStats: true
{% endhighlight %}

![updated homepage](/uploads/2024-12-05-homelab1/homepage-updated.png)

There's not much to show right now, but at least everything is set up and working. I'll probably add more services as I find a need for them, but for now, I'm happy with what I have.

## File Sharing

This is a service that I know I'm going to use a lot. For the past few years, I've had student access for OneDrive, and it's given me effectively infinite cloud storage. But that's going to end soon, and I'm not going to be paying for a subscription. My dad uses a Synology NAS and I can use it, but it's been a pain to use and I'm not going to mess with it. My dad has his own way of using it, but it doesn't work for me. So I'm going to set up a file sharing service on my server.

My first thought was to use Samba, as it's something I've used before, but it's not containerised, and I'd rather stick with containers for ease of management. There are containers for it, but I'd rather not use them. Samba itself is very simple, but deviating from default settings can be a pain. I'd rather use something that's more user-friendly. I usually look for services on [awesome-selfhosted][awesome-selfhosted-link], and I found a few services that run on docker. Nextcloud and Owncloud are the big ones, but they both feel a bit overkill for what I need. Bewcloud is a new and supposedly simpler alternative, but it's still quite new and it's only got one developer, so its future is a bit uncertain.

There's also Seafile and Filerun, which are just for file syncing. They're much less comprehensive, but both sound good enough for my needs. I ultimately decided to go with Seafile since it seems more well-known.

Again, the [documentation][seafile-docs] was pretty straightforward, with the only change I made being to use port 8001 instead of 80. I was able to access the Seafile web UI from my desktop using the hostname, and I was able to set up the Seafile client on my desktop and sync files between the server and my desktop.

However there is an issue where file previews in the web UI doesn't work and downloading files from the web UI doesn't work. I'll have to look into this later when I have the energy.

<hr>

Hero image by [Kirill Sh](https://unsplash.com/@kirill2020) on [Unsplash](https://unsplash.com/)

[docker-engine-docs]: https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
[portainer-install-docs]: https://docs.portainer.io/start/install-ce/server/docker/linux
[homepage-link]: https://github.com/gethomepage/homepage
[lvm-extend]: https://packetpushers.net/blog/ubuntu-extend-your-default-lvm-space/
[awesome-selfhosted-link]: https://awesome-selfhosted.net/
[seafile-docs]: https://manual.seafile.com/12.0/setup/setup_ce_by_docker/
