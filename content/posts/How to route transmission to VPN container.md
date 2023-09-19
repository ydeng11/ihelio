---
title: "How to route transmission to VPN container?"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - Home Server
tags: 
    - NAS
    - Docker
    - VPN
    - Torrenting
---

# Intro
When you have a NAS at home, it feel bad if you don't keep it running for something even if you don't use it. It is the major backup to store the photos we shoot with our iphones. However, it is just basic use and hosting a media center with it sounds more cool. I used to use [transmission-openvpn](https://github.com/haugene/docker-transmission-openvpn) and it works perfectly, however, my VPN expires and I decided to use proton free tier. This is where things go south - I spent two days on it and I cannot get it work. When I am about to give up, I found a better approach for my purpose - running transmission and vpn in separate docker container and route transmission to vpn conatiner. It sounds a great idea in the first place given we should only do one job for each service - unix's princinple.

Now I will go through how I set it up and how it works for me in the following sections.
# Tools
Though everything could be handled by docker command or docker app in synology, I highly suggest to use a container management platform - portainer. After installing portainer, we are good to go to install vpn container and download client.

## Portainer
This [article](https://mariushosting.com/how-to-install-portainer-on-your-synology-nas/) illustrates how to install portainer on synology so I will just put it here to avoide deplicate efforts. **Note I am using portainer for Docker**.

## VPN Container
There are tons of choices and each provider should have its own docker image to use. But what I use is [gluetun](https://github.com/qdm12/gluetun) which is a lightweight vpn client providing all the functions you need and has tremendous support to different providers. 

To install gluetun, we need find the setup guide for the provider we use. Since I am using proton VPN so I will use this [guide](https://github.com/qdm12/gluetun/wiki/ProtonVPN). Remember portainer supports multiple ways of deploying container and I prefer to deploy it via `Stacks` which is actually via the compose file.  
![](https://i.imgur.com/s1EsNrT.png)

After clicking on `Add Stack`, there is a web editor we could type in our compose file like
![](https://i.imgur.com/aYxCx9r.png)
Remember proton VPN has different user/password for connecting openVPN which is not the one we use to login. I was trapped there when I was working on transmission-openvpn container and I don't want others waste their time on it. It is also noted we don't have any port mapping here. We will set up port mapping later for better visibility.

After we have gluetun installed, our VPN container should be good to go. And we could check the log to verify if we have it connected as shown below.
```
2022-09-30T00:05:11Z INFO [ip getter] Public IP address is 143.244.44.185 (United States, New York, New York City)

```

## Downloadclient
Similarly, there are tons of download clients you could use. qBitorrent and transmission are two widely used clients. qBitorrent has more functions and prob better UI, and transmission is very lightweight and concise. So it is really your call. I used transmission for a while and I tend to keep using it. 

Same way as installing gluetun, we just need get the compose file and mine is 
```yaml
---
version: "2.1"
services:
  transmission:
    image: lscr.io/linuxserver/transmission:latest
    container_name: transmission
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=USA/NEW YORK
    volumes:
      - /volume1/docker/transmission/config:/config
      - /volume1/docker/transmission/downloads:/downloads
      - /volume1/docker/transmission/watch:/watch
    restart: unless-stopped
```

So now we have all the tools we need: portainer, gluetun and transmission. The last part is how we route transmission to gluetun.

# Routing
You will be surprised how simple it is now. First we need setup port mapping in gluetun. According to the [transmission documentation](https://hub.docker.com/r/linuxserver/transmission), the port exposed by transimission is `-p 9091:9091 -p 51413:51413 -p 51413:51413/udp`. We just need make the same port mapping in **gluetun** as
![](https://i.imgur.com/4ixrJ4L.png)

The final step is to modify the **network** in **transmission container**, we go to transmission container and click on `duplicate/edit`, and setup network in advanced conatiner settings as:
![](https://i.imgur.com/pU2vgqm.png)

*Because we are using the container as the network of transmission, we need do the port mapping in the glutun container instaed of transmission container.* 

After deploy it again, the transmission will be routed to gluetun. To verify that, we can check the ip address of transmission by ssh to the container:
![](https://i.imgur.com/ZksWgD3.png)
The ip address is the same as what we saw from gluetun.

# Conclusion
Now everything is ready and we are good to use transimission now. I tried the speed and it looks good to me. I am going to run with this gig for a while hopefully proton free is sufficient for me. 

---
# References
https://mariushosting.com/how-to-install-portainer-on-your-synology-nas/
https://www.youtube.com/watch?v=vUyHGF1HMsw&t=495s
https://www.youtube.com/watch?v=xbSfaKwyfXE