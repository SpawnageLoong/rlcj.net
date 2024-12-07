---
layout: post
date: 2024-12-07 12:00:00 +0800
title: 'Getting Cloudflare Tunnels to Work'
tags:
  - homelab
hero: /uploads/2024-12-07-homelab3/hero.jpg
overlay: blue
---

Let's continue from where I left off in the last post. I'm attempting to set up remote access to my homelab's services and so far I haven't been successful. Check out my previous post [here]({% post_url 2024-12-06-homelab-security %}) if you haven't already. I'm going to start today's attempts by changing my existing services to use traefik as a reverse proxy.

## Cloudflared

Before going through the effort of moving _everything_, I should probably test if my idea has any merit at all. My current plan is to use Cloudflare Tunnels to expose my services to the internet, and the way that works is by running the `cloudflared` container in my Docker instance, giving cloudflare access to the default internal Docker network (at least that's my current understanding).

The logs so far have been showing me that the addresses the tunnel is trying to reach cannot be resolved as they aren't using my PiHole as the DNS server. I'm not too sure how to fix this issue, so I'm trying out a few things and seeing if anything sticks.

First thing I did was stop (not remove) the existing `cloudflared` container using Portainer, then setting up a `docker-compose.yml` for the new `cloudflared` container.

{% highlight yaml %}
services:
    cloudflared-traefik:
        image: cloudflare/cloudflared:latest
        container_name: cloudflared-traefik
        dns: 192.168.1.112
        command: tunnel --no-autoupdate run --token
            TOKEN_GOES_HERE
        networks:
            - proxy
        restart: unless-stopped
networks:
  proxy:
    external: true
{% endhighlight %}

Then I went into Cloudflare's dashboard and created a new tunnel for me to work with and got the token for it. I then copied the token into the `docker-compose.yml` file and deployed the stack using Portainer. I just copy-paste the `docker-compose.yml` into the `Stacks` section of Portainer and hit `Deploy the stack`.

Now when I try and access the URL that I set in the dashboard, I get a `404 page not found error`, which at least tells me I'm doing something different from last time. A quick check of the `cloudflared-traefik` logs shows that there were no errors, unlike before where I was getting a lot of `DNS resolution failed` errors. This should mean that I'm on the right track.

A quick google search for "traefik cloudflare tunnel 404" got me an immediate solution (for once) on the first result which was a [reddit post][reddit-post] that told me to add a HTTP Host header in cloudflare's dashboard. I added the header and now it works! I can access the nginx hello page from the internet!

![nginx hello page](/uploads/2024-12-07-homelab3/nginx.png)

## Moving everything behind Traefik

Now that I know I can get the tunnel working, it's time to move everything I can behind Traefik. Right now that's just homepage and portainer, but this should get me familiar enough with the process to make future services easier to set up.

First off, Portainer. I created a new `docker-compose.yml` file for Portainer and added the following:

This is not the working docker-compose file. The working one is later in the post.
{:.notice-alert}

{% highlight yaml %}
name: portainer
services:
    portainer-ee:
        image: portainer/portainer-ee:2.21.4
        container_name: portainer
        restart: always
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - portainer_data:/data
        networks:
            - proxy
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.nginx.rule=Host(`ADDRESS_HERE`)"
            - "traefik.http.routers.nginx.entrypoints=https"
            - "traefik.http.routers.nginx.tls=true"
            - "traefik.http.services.nginx.loadbalancer.server.port=9443"
volumes:
    portainer_data:
        external: true
        name: portainer_data
networks:
  proxy:
    external: true
{% endhighlight %}

There was a compose file available on Portainer's GitHub, but that was last updated 5 years ago, so I didn't want to use that as a basis. I also had to add the labels for portainer to work with traefik, which I copied from the nginx example. I then went into PiHole and added a CNAME record for the address I was going to use.

Note that I'm installing Portainer Business Edition now instead of the Community Edition. I got my free license renewed, so I figured why not?
{: .notice}

Then I went ahead and removed my previous portainer container using `docker stop portainer` and `docker rm portainer`. A quick `docker compose up -d --force-recreate` later and the new container was running. Except it's not accessible anymore. I'm getting a `404 page not found` error again, even though it shouldn't be going through a tunnel. And my nginx-hello page is also gone now, replaced with a `404 page not found` error.

The traefik dashboard is still accessible though, and it's showing that the nginx-hello rule is missing. I'm not sure why that is, but I'm going to try and fix that first. I stopped the portainer container and the nginx-hello container rule reappeared. I'm guessing there's a conflict between the two services, so I'll go back and have alook at the labels that I copied that might be causing the issue. Then I realised all of the labels were named `nginx` instead of `portainer`. I changed the labels to `portainer`, reran the compose command, aaaaaaaand it's still not working.

{% highlight yaml %}
name: portainer
services:
    portainer-ee:
        image: portainer/portainer-ee:2.21.4
        container_name: portainer
        restart: always
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - portainer_data:/data
        networks:
            - proxy
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.portainer.rule=Host(`ADDRESS_HERE`)"
            - "traefik.http.routers.portainer.entrypoints=https"
            - "traefik.http.routers.portainer.tls=true"
            - "traefik.http.services.portainer.loadbalancer.server.port=9443"
volumes:
    portainer_data:
        external: true
        name: portainer_data
networks:
  proxy:
    external: true
{% endhighlight %}

At least now the rule shows up in traefic.

![traefik dashboard](/uploads/2024-12-07-homelab3/traefik.png)

A few refreshes of the portainer page later, and the error changed to `Client sent an HTTP request to an HTTPS server.`. I managed to find a StackOverflow [post][stackoverflow-post] that told me that port 9000 is portainer's http port, so I changed the port to 9000 in the labels and now it works! I'm back in portainer! And since I never deleted the old portainer volume, all my settings are still there and I can skip going through the initial setup again.

## Homepage

I'm going to move homepage behind traefik now, but this time I'm going to change things up a bit. Instead of one homepage, I'm going to run two instances of homepage, one for the internal network and one for the external network. This way I can have a different homepage for when I'm at home and when I'm not. Certain services like PiHole should not be accessible outside of my local network, so no point in having visible when I'm away.

First I removed the existing homepage container and created a new `docker-compose.yml` file for homepage.
I started by copying the default compose file from homepage's [documentation][homepage-install], and pasting it twice in the same file. I changed the container names and ports for the two instances, added the paths for each of the config folders, added them to the `proxy` network, and added the traefik labels.

This is not the working docker-compose file. The working one is later in the post.
{:.notice-alert}

{% highlight yaml %}
name: homepages
services:
  homepage-internal:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage-internal
    ports:
      - 3000:3000
    volumes:
      - /home/richard/container-data/homepage/internal-config:/app/config
      - /var/run/docker.sock:/var/run/docker.sock # (optional) For docker integrations
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.homepage-internal.rule=Host(`ADDRESS_HERE`)"
      - "traefik.http.routers.homepage-internal.entrypoints=https"
      - "traefik.http.routers.homepage-internal.tls=true"
      - "traefik.http.services.homepage-internal.loadbalancer.server.port=3000"

  homepage-external:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage-external
    ports:
      - 3001:3000
    volumes:
      - /home/richard/container-data/homepage/external-config:/app/config
      - /var/run/docker.sock:/var/run/docker.sock # (optional) For docker integrations
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.homepage-external.rule=Host(`ADDRESS_HERE`)"
      - "traefik.http.routers.homepage-external.entrypoints=https"
      - "traefik.http.routers.homepage-external.tls=true"
      - "traefik.http.services.homepage-external.loadbalancer.server.port=3001"
networks:
  proxy:
    external: true
{% endhighlight %}

I added the two addresses to PiHole and deployed the stack using Portainer. The internal homepage works, but the external one doesn't. I'm getting a `bad gateway` error. I figured this happened to me before when I changed to ports when I first set up traefik. I removed the ports for both instances and changed the rule back to port 3000 and it works! I don't need to expose any of the container's ports since traefik handles that for me.

I was getting `ERR_CERT_AUTHORITY_INVALID` warnings for the external homepage though, as the certificates were showing up as `TRAEFIK DEFAULT CERT`. Traefik was supposed to get the certificates from Let's Encrypt, but it wasn't doing that. Then I realised it's because of the domain. My traefik setup only got me certs for the `*.local.rlcj.net` domain, but that wasn't the domain I was using for the external homepage. I changed the domain in the traefik labels and in PiHole, and now the certs are back.

I didn't copy my previous homepage config files, so now I have to customise each homepage again. Starting with the internal homepage, I added all of the services I had running (Portainer, PiHole, Traefik and a link to the external homepage), added the widget to the services that support it, and added shortcuts to my usual bookmarked websites.

![internal homepage](/uploads/2024-12-07-homelab3/homepage.png)

For now I don't have any services running on the external homepage, but I'll add them as I need them. I called it a day here, and next time I'll be working on Nextcloud.

<hr>

[reddit-post]: https://www.reddit.com/r/Traefik/comments/w6cckh/traefik_cloudflare_tunnel_not_working_perfectly/
[stackoverflow-post]: https://stackoverflow.com/questions/72583330/portainer-client-sent-an-http-request-to-an-https-server-despite-the-url-is-h
[homepage-install]: https://gethomepage.dev/installation/docker/
