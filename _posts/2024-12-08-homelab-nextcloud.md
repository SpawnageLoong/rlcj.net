---
layout: post
date: 2024-12-08 12:00:00 +0800
title: 'Setting up Nexcloud on my Homelab'
tags:
  - homelab
  - nextcloud
hero: /uploads/2024-12-08-homelab-nextcloud/hero.jpg
overlay: orange
---

It's finally time to add some "useful" services to my Homelab. Everything so far has been about managing the server "infrastructure", but now it's time to add services that I actually intend to use on a regular basis. The first service I'm going to add is Nextcloud.

Nextcloud is a file sync service that works similar to Google Drive or OneDrive. As I mentioned before, I currently use OneDrive because my school provides a free subscription for students, but with graduation coming soon, I need to get an alternative in place. I chose nextcloud mainly for its relatively large userbase, so I can find help if I run into issues.

## I make a lot of typos

I'm going to be using the [Nextcloud AIO docker image][nextcloud-aio]. This image is a single container that contains everything needed to run Nextcloud. While they provide specific instructions for how to adapt the image to run behind a reverse proxy, like I've got in my setup, I'm going to use the process I've been using for all my other services. I'll run the container in the `proxy` network and use Traefik reverse proxy to route traffic to it. That way I shouldn't have to worry about ports, SSL certificates, etc.

Nextcloud AIO provides a compose file that I can use to get started. I'll make a few modifications to the file to fit my setup, but it should be pretty straightforward.

This is not the working docker-compose file. The working one is later in the post.
{:.notice-alert}

{% highlight yaml %}
services:
  nextcloud-aio-mastercontainer:
    image: nextcloud/all-in-one:latest
    container_name: nextcloud-aio-mastercontainer # This line is not allowed to be changed as otherwise AIO will not work correctly
    init: true
    restart: always
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config # This line is not allowed to be changed as otherwise the built-in backup solution will not work
      - /var/run/docker.sock:/var/run/docker.sock:ro # May be changed on macOS, Windows or docker rootless. See the applicable documentation. If adjusting, don't forget to also set 'WATCHTOWER_DOCKER_SOCKET_PATH'!
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nextcloud-aio.rule=Host(`ADDRESS_HERE`)"
      - "traefik.http.routers.nextcloud-aio-internal.entrypoints=https"
      - "traefik.http.routers.nextcloud-aio-internal.tls=true"
      - "traefik.http.services.nextcloud-aio-internal.loadbalancer.server.port=8080"
volumes: # If you want to store the data on a different drive, see https://github.com/nextcloud/all-in-one#how-to-store-the-filesinstallation-on-a-separate-drive
  nextcloud_aio_mastercontainer:
    name: nextcloud_aio_mastercontainer # This line is not allowed to be changed as otherwise the built-in backup solution will not work
networks:
  proxy:
    external: true
{% endhighlight %}

The main changes I've made are to remove the ports section since Traefik will handle that, set the container to use the `proxy` network, and to add the labels for traefik. After this, I also need to add the address I'm using to the PiHole's local DNS.

Again, I used Portainer to create the stack, and after the container started up, I tried accessing the web UI. And it gave me a `404 page not found` error. I checked the traefik dashboard and found two rules for nextcloud rather than the one that the other services have.

![Traefik Dashboard](/uploads/2024-12-08-homelab-nextcloud/traefik-dashboard.png)

Then I realised I'd derped and made a few typos in the compose file because of careless `Ctrl+F + Replace` usage. I did the first rule by hand, then used `replace` to replace `homepage` with `nextcloud-aio`, but forgot to replace the "-internal". After removing the "-internal"s, the web UI loaded up correctly.

This is still not the working docker-compose file, witness my great typo-brain. The working one is still later in the post.
{:.notice-alert}

{% highlight yaml %}
services:
  nextcloud-aio-mastercontainer:
    image: nextcloud/all-in-one:latest
    container_name: nextcloud-aio-mastercontainer # This line is not allowed to be changed as otherwise AIO will not work correctly
    init: true
    restart: always
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config # This line is not allowed to be changed as otherwise the built-in backup solution will not work
      - /var/run/docker.sock:/var/run/docker.sock:ro # May be changed on macOS, Windows or docker rootless. See the applicable documentation. If adjusting, don't forget to also set 'WATCHTOWER_DOCKER_SOCKET_PATH'!
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nextcloud-aio.rule=Host(`nextcloud.local.rlcj.net`)"
      - "traefik.http.routers.nextcloud-aio.entrypoints=https"
      - "traefik.http.routers.nextcloud-aio.tls=true"
      - "traefik.http.services.nextcloud-aio.loadbalancer.server.port=8080"
volumes: # If you want to store the data on a different drive, see https://github.com/nextcloud/all-in-one#how-to-store-the-filesinstallation-on-a-separate-drive
  nextcloud_aio_mastercontainer:
    name: nextcloud_aio_mastercontainer # This line is not allowed to be changed as otherwise the built-in backup solution will not work
networks:
  proxy:
    external: true
{% endhighlight %}

I gave the container a few seconds to restart, then tried accessing the web UI again. This time it took a while, but eventually failed with a `Gateway Timeout` error. I checked the traefik dashboard again and this time there was only one rule, which was what I expected. I checked the service, and noticed the URL it was pointing to was `http://172.19.0.2:8080`, which seemed slightly off to me. I went back to portainer and realised the container was not in the `proxy` network. I had managed to forget to add the network to the compose file. Another fix later, and the container was in the network.

This is _still_ not the working docker-compose file.
{:.notice-alert}

{% highlight yaml %}
services:
  nextcloud-aio-mastercontainer:
    image: nextcloud/all-in-one:latest
    container_name: nextcloud-aio-mastercontainer # This line is not allowed to be changed as otherwise AIO will not work correctly
    init: true
    restart: always
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config # This line is not allowed to be changed as otherwise the built-in backup solution will not work
      - /var/run/docker.sock:/var/run/docker.sock:ro # May be changed on macOS, Windows or docker rootless. See the applicable documentation. If adjusting, don't forget to also set 'WATCHTOWER_DOCKER_SOCKET_PATH'!
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nextcloud-aio.rule=Host(`nextcloud.local.rlcj.net`)"
      - "traefik.http.routers.nextcloud-aio.entrypoints=https"
      - "traefik.http.routers.nextcloud-aio.tls=true"
      - "traefik.http.services.nextcloud-aio.loadbalancer.server.port=8080"
volumes: # If you want to store the data on a different drive, see https://github.com/nextcloud/all-in-one#how-to-store-the-filesinstallation-on-a-separate-drive
  nextcloud_aio_mastercontainer:
    name: nextcloud_aio_mastercontainer # This line is not allowed to be changed as otherwise the built-in backup solution will not work
networks:
  proxy:
    external: true
{% endhighlight %}

This time, it's giving me a `Bad Request` error page.

![Nextcloud Bad Request](/uploads/2024-12-08-homelab-nextcloud/bad-request.png)

I copy-pasted the whole error into duckduckgo (yes I switched to duckduckgo at this point of time) and got a list of results that all talked about different things. So I put that to the side and tried a more unga-bunga approach, which is to look at each label and see what changes I could make and just try that. A "throw things at the wall and see what sticks" approach. I started with the port, changing from 8080 to 8443. That gave me a different error: `Client sent an HTTP request to an HTTPS server.`.

Then I remembered that the Nextcloud AIO GitHub page has guides for the changes needed for the various reverse proxies, so I went back to that page to look for the changes that might be needed. I tried changing the port to 11000 since that's the default Apache port, but that gave me a `Bad Gateway` error. Then I started copying the environment settings that were commented out in the original compose file, and changed them to match my own domain, and set the port back to 8080. This time, I was back at the Bad Request error page.

Up until this point, I hadn't realised the labels I was using corresponded to the the config files that was in the reverse proxy instructions. After a few google searches, I realised this and went back to the AIO gihub page to look at the config file and started copying the labels. I restarted the container, but still no change. The traefik dashboard did reflect the changes I made, but the web UI was still giving me the Bad Request error.

## Abandoning the AIO image

After a few more searches with terms like "nextcloud traefik 3", I found a [post][dev-post] on dev.to that opted to set up individual services rather than using the AIO image. Unlike the previous attempts, I opted to copy the whole compose file and make changes only as indicated by the author (things like the domain for the router rule). This compose file already had the network names `proxy`, so that could be left alone, but I did change to path to the folder which would act as a volume for the containers.

This still didn't work, and now I was getting a `Gateway Timeout` error. I went into traefik to have a look at what was going on, and it looked like the service was trying to use port 80 on the `nextcloud_default` network. I went back into the compose file and changed the port from 80 to 8080, but that didn't seem to do anything. So I tried changing all of the containers to use the `proxy` network only. Now the error became `Bad Gateway`. I changed the port back to 80, and now the nextcloud login UI was showing up!

{% highlight yaml %}
services:
    nextcloud-db:
        image: mariadb:latest
        container_name: nextcloud-db
        command: --transaction-isolation=READ-COMMITTED --log-bin=ROW --skip-innodb-read-only-compressed
        restart: unless-stopped
        volumes:
            - /home/richard/container-data/nextcloud/mysql-data:/var/lib/mysql
        environment:
            - MYSQL_ROOT_PASSWORD=PASSWORD_GOES_HERE # MYSQL ROOT PASSWORD
            - MYSQL_PASSWORD=PASSWORD_GOES_HERE # PASSWORD OF MYSQL USER
            - MYSQL_DATABASE=nextcloud # MYSQL DATABASE
            - MYSQL_USER=nextcloud # MYSQL USERNAME
        networks:
            - proxy

    nextcloud-redis:
        image: redis:alpine
        container_name: nextcloud-redis
        hostname: nextcloud-redis
        restart: unless-stopped
        command: redis-server --requirepass PASSWORD_GOES_HERE # REPLACE WITH REDIS PASSWORD
        networks:
            - proxy

    nextcloud-app:
        image: nextcloud
        container_name: nextcloud-app
        restart: unless-stopped
        depends_on:
            - nextcloud-db
            - nextcloud-redis
        environment:
            - REDIS_HOST=nextcloud-redis
            - REDIS_HOST_PASSWORD=PASSWORD_GOES_HERE # REPLACE WITH REDIS PASSWORD
            - MYSQL_HOST=nextcloud-db
            - MYSQL_USER=nextcloud # MYSQL USERNAME
            - MYSQL_PASSWORD=PASSWORD_GOES_HERE # PASSWORD OF MYSQL USER
            - MYSQL_DATABASE=nextcloud # MYSQL DATABASE
        volumes:
            - /home/richard/container-data/nextcloud/nextcloud:/var/www/html
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.nextcloud.rule=Host(`nextcloud.local.rlcj.net`)"
            - "traefik.http.routers.nextcloud.entrypoints=http"
            - "traefik.http.middlewares.https-redirect.redirectscheme.scheme=https"
            - "traefik.http.routers.nextcloud.middlewares=nc-header,https-redirect"
            - "traefik.http.routers.nextcloud-secure.entrypoints=https"
            - "traefik.http.routers.nextcloud-secure.rule=Host(`nextcloud.local.rlcj.net`)"
            - "traefik.http.middlewares.nc-rep.redirectregex.regex=https://(.*)/.well-known/(card|cal)dav"
            - "traefik.http.middlewares.nc-rep.redirectregex.replacement=https://$$1/remote.php/dav/"
            - "traefik.http.middlewares.nc-rep.redirectregex.permanent=true"
            - "traefik.http.middlewares.nc-header.headers.customFrameOptionsValue=SAMEORIGIN"
            - "traefik.http.middlewares.nc-header.headers.customResponseHeaders.Strict-Transport-Security=15552000"
            - "traefik.http.routers.nextcloud-secure.middlewares=nc-rep,nc-header"
            - "traefik.http.routers.nextcloud-secure.tls=true"
            - "traefik.http.routers.nextcloud-secure.tls.certresolver=letsencrypt"
            - "traefik.http.routers.nextcloud-secure.service=nextcloud"
            - "traefik.http.services.nextcloud.loadbalancer.server.port=80"
            - "traefik.http.services.nextcloud.loadbalancer.passHostHeader=true"
        networks:
            - proxy
            # - default

networks:
  proxy:
    external: true
{% endhighlight %}

![Nextcloud Login](/uploads/2024-12-08-homelab-nextcloud/nextcloud-login.png)

I skipped installing any NextCloud apps for now.
{:.notice}

Now that I could access NextCloud, I set up the admin account and headed into the administration settings, where there were a list of errors and warnings regarding the setup. Some looked easy to fix, and some I had no clue about:

- The reverse proxy header configuration is incorrect.
- Accessing site insecurely via HTTP. You are strongly advised to set up your server to require HTTPS instead.
- Server has no maintenance window start time configured.
- One or more mimetype migrations are available.
- Some headers are not set correctly on your instance - The `Strict-Transport-Security` HTTP header is malformed
- Detected some missing optional indices.
- MariaDB version "11.6.2-MariaDB-ubu2404-log" detected. MariaDB >=10.6 and <=11.4 is suggested for best performance, stability and functionality with this version of Nextcloud.
- Your installation has no default phone region set.
- You have not set or verified your email server configuration, yet.

Some of these issues I can probably ignore, but for my own peace of mind, I'll try to fix all of them. I started at the bottom and worked my way up, since the ones at the bottom seemed the easiest to fix.

## Fixing the issues

### Email server configuration

I ended up skipping this because I don't have an email server right now, so I'll get back to this when I set one up.

### Default phone region

This was simple enough, I just had to open the config.php file in my nextcloud folder (the one mapped to a volume) and add the following line:

{% highlight php %}
'default_phone_region' => 'SG',
{% endhighlight %}

### MariaDB version

This was also simple, I just went to the docker-compose file and changed the image tag to `11.4`, then restarted the stack. I checked the Nextcloud admin page again, and the warning for the phone region and MariaDB was gone.

### Missing optional indices

This involved going into the docker container for the Nextcloud app and using nextcloud's `occ` command to add the missing indices. I used the following command to to run the program in the container (this command was run from outside the container):

{% highlight bash %}
docker exec -it -u 33 nextcloud-app /var/www/html/occ db:add-missing-indices
{% endhighlight %}

Refreshing the admin page showed that the warning was gone.

### Strict-Transport-Security header

A quick search for "Strict-Transport-Security" led me to the [MDN Web Docs page][strict-transport-security-mdn] for the header, which showed mw that the malformed header was due to the value being set to `15552000`. I went back to the compose file and changed the value to `max-age=15552000`. After restarting the stack, the warning was gone.

### Mimetype migrations

I have no clue what mimetypes are, but the warning did provide me with a command to run to fix the issue. Did the same thing as before and ran the following command outside the container:

{% highlight bash %}
docker exec -it -u 33 nextcloud-app /var/www/html/occ maintenance:repair --include-expensive
{% endhighlight %}

After running the command, the warning was gone.

### Maintenance window start time

After checking the docs link the warning gave me, I found that I could set the maintenance window start time by running the following command:

{% highlight bash %}
docker exec -it -u 33 nextcloud-app /var/www/html/occ config:system:set maintenance_window_start --type=integer --value=19
{% endhighlight %}

This sets the maintenance mode to start at 7pm UTC (which is 3am UTC+8), and it lasts for 4 hours by default. The warning was gone after running the command.

### Set requirement for HTTPS

This is something I was a bit confused about, since I thought traefik was handling the HTTPS part. I went back to the compose file and changed the entrypoint for the `nextcloud` router to `https`, and added `;includeSubDomains;preload` to the strict-transport-security header as per nextcloud's recommendation in the docs. I restarted the stack, but the warning was still there.

I wasn't too sure how to fix this, so I put it on hold for a bit and went on the the last warning.

### Reverse proxy header configuration

Based on the docs link provided by the warning, I needed to specify the IP address of the reverse proxy by adding the following lines to the config.php file, where the IP address was the IP of the reverse proxy on the `proxy` network:

{% highlight php %}
'trusted_proxies'   => ['IP_ADDRESS_HERE']
{% endhighlight %}

I restarted the stack, and the warning was gone.

### Back to HTTPS

I figured this is an issue in traefik rather than in Nextcloud, so I went back to the traefik dashboard to see if I could find anything that might help. The labels in the compose file provide two entry points to access NextCloud: `nextcloud` and `nextcloud-secure`. `nextcloud` is the entrypoint responsible for redirecting to HTTPS, so I tried commenting out all the labels for it and restarting the stack.

This didn't remove the warning, but the routers in the Traefik dashboard changed, with the `nextcloud` router being removed, but a new router for `nextcloud-app-nextcloud` being added. There was also a new warning in the Nextcloud admin page: `Last background job execution ran 1 hour ago. Something seems wrong.`. This was an issue with `cron`, but I can put it off for now.

I went back to look at the compose file, and noticed the load balancers were under `nextcloud` ratehr than `nextcloud-secure`, so I uncommented the labels for `nextcloud` and moved the load balancers to `nextcloud-secure`. Whoops, that broke the router in traefik, so let's undo that.

My next attempt was to comment out everything related to http, and rename `nextcloud-secure` to `nextcloud`, which should leave only https available for routing. While this did work in removing http in the traefik dashboard, the warning was still there.

Now I was trying to throw things at the wall again, by commenting out the cert resolver and the service specifier, leaving these things for traefik to figure out on its own. After restarting the stack, the warning was still there.

Even more searching later, I found this [post][nextcloud-forum-post] on the Nextcloud forums that highlighted this line in the config.php file:

{% highlight php %}
"overwritecondaddr": "^127\\.0\\.0\\.1$",
{% endhighlight %}

I didn't have this line in my config.php file, but I did see an `overwrite.cli.url` line that was set to `http://nextcloud.local.rlcj.net`. I changed this to `https://nextcloud.local.rlcj.net`, and restarted the stack, but this still didn't remove the warning. Another forum post suggested changing the Overwrite parameters in the config.php file, but I didn't have any of those in my file, so I added the following:

{% highlight php %}
'overwriteprotocol' => 'https',
{% endhighlight %}

And somehow that worked. The warning was gone.

I went back to the docker compose file and uncommented the labels for the `nextcloud` router so that http traffic would be redirected to https. After restarting the stack, the warning was still gone, and I could access the Nextcloud web UI without any issues.

### Background job execution

Now for the newest error to appear. This appears to be an issue with the cron job not running. After a bit of searching on the forums, it seems like you need to set up an entire container running nextcloud just for the cron job, or you have to modify your existing container. I decided that it wasn't worth the effort to figure it out, so I just went into the Nextcloud settings and changed the background jobs to use AJAX instead.

### Error in logs

After fixing all the warnings and errors in the admin page, I went to check the logs to see if there were any errors there. There was one error that kept appearing:

`Undefined array key "error" at /var/www/html/apps/weather_status/lib/Service/WeatherStatusService.php#260`

I checked the file in question and it looked like an error I could ignore as it relates to the weather app, which was still working normally. Unfortunately, I couldn't find a way to disable the error message, so I'll just have to live with it.

## Testing

Now that everything was working on the local network, it was time to install the clients on my devices and test the sync functionality. I installed the Nextcloud app on my desktop, and set up the sync folder. I created a folder called `Test Folder` and copied a 1.5GB video into it. The NextCloud client initially threw an error saying the file wasn't found, but that seemed to be due to copying the video before the folder had been synced. I tried using the manual "sync now" button and it worked fine. I refreshed the web UI and the video was there and playable. I then tried accessing the video on my phone app, and it worked fine there as well.

## Remote Access

Now that everything worked locally, it was time to make it accessible from outside my home. I added Nextcloud to my existing Cloudflare Tunnel along with the external homepage, and now both of them are accessible from anywhere. I did notice that nextcloud was automatically redirecting to the local network address rather than staying on the external address. I tried to fix it by removing the `overwrite.cli.url` parameter in the config.php file, but that didn't seem to work.

Anyway before I continue to mess with that, I want to add an extra layer of security to my external services by preventing unauthorised users from even seeing the login pages of my outward facing services. I'll be using Cloudflare Zero Trust for this, and it's easy enough to set up. You just have to create an access group (in this case it's just myself), add the URLs you want to the application you're protecting, and then add the group to the application. After that, you can set the application to "Block" for all other users, and only you will be able to access the login pages.

In my case, you'll first see a Cloudflare login page, and then the login page for the service. This is a bit annoying, but it's better than having the login page accessible to anyone. This way unauthorised users won't be able to contact my homelab, even if they can guess the URL.

![Cloudflare Login](/uploads/2024-12-08-homelab-nextcloud/cloudflare-login.png)

Now that Zero Access was set up, I tried to access Homepage and Nextcloud from my phone using mobile data, and while homepage was working fine, Nextcloud wasn't because of the redirect issue. I tried to fix it by adding this line to the config.php file:

{% highlight php %}
'overwritehost' => 'EXTERNAL_ADDRESS_HERE',
{% endhighlight %}

This doesn't really fix the issue, but it does make the web UI accessible from the external address. Now the forced redirect instead redirects to the external address, which works well enough. Then I realised the Cloudflare Access page also affects the Nextcloud app, so I had to remove access from the Nextcloud app. In the future I'll have to buy a separate domain for my non-public homelab services, so I can do the "security through obscurity" thing.

I ran my server through the [Nextcloud Security Scan][nextcloud-security-scan] where it scored an A+, and the [Mozilla Observatory][mozilla-observatory] where it also got an A+ (I didn't even know you could get 115/100), which I'm pretty happy about.

<hr>

This has been a long post, but I'm glad I managed to get Nextcloud up and running. I can now copy some files from my OneDrive into Nextcloud, and evaluate how well it works for me.

<hr>

[nextcloud-aio]: https://github.com/nextcloud/all-in-one
[dev-post]: https://dev.to/fabiancdng/running-nextcloud-using-docker-and-traefik-3-1k2j
[strict-transport-security-mdn]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
[nextcloud-forum-post]: https://help.nextcloud.com/t/ssl-works-fine-but-i-am-still-getting-accessing-site-insecurely-via-http-you-are-strongly-advised-to-set-up-your-server-to-require-https-instead-error/192921
[nextcloud-security-scan]: https://scan.nextcloud.com/
[mozilla-observatory]: https://observatory.mozilla.org/
