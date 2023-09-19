---
title: "Build a home media server but automated"
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

## Intro

In [Route transmission to VPN container](https://ihelio.today/archives/howtoroutetransmissiontovpncontainer), I talked about how to download contents via VPN tunnel so we can get rid of some troubles. But that is far from enough for us to build a home media server which should work as Netflix and Hulu to us. We shouldn't bother with the torrent and subtitle search. Once we add a show to our watch list, everything should be set up automatically. I will introduce the tools and necessary setup for me to build a home media server in the following sections.

## Structure

![Structure](https://i.imgur.com/ZaCj4nu.png)

All the above services are running inside a docker box which could be synology and rasperry Pi. Take Sonarr as an example:
- Once we add a show to Sonarr, it will search the sees via Jackett
- Then Sonarr will add the download tasks to the downloadclient like transmission in my deployment
- Then the contents will be downloaded to local path
- Then Plex could genear metadata from the configred folder
- The generated metata will assis ChineseSubFinder to download subtitles (Optional)
The workflow applies to the radarr as well. Thus we could enjoy the show by just adding it to our library without worrying the seeds and subtitles.

## Configuration
### Portainer
I use `portainer` to manage the docker which provides more flexibility than docker app in Synology. I highly recommend everyone to use it to manage the containers.
### VPN & Download Client
In [Route transmission to VPN container](https://ihelio.today/archives/howtoroutetransmissiontovpncontainer), I used proto VPN free version but I upgraded to NordVPN with the Christmas deal. But proto VPN free version is sufficient if you only need for downloading contents. 

Everything else is the same as I am still using gluetun to connect the VPN and handle the traffic from other containers. If you wonder how to route the download client to VPN tunnel, please check that article since we need do the same thing for other containers.
### Jackett, Sonarr and Radarr
After installing the three containers in `portainer`,  it is preferred to use the VPN as an extra layer so I routed `Jackett` to VPN container (e.g. gluetun) as well.

As Sonarr and Radarr, routing them to VPN will confuse them to connect to `Jackett` and the download client. So we just need deploy them as default setting (remember to change the volume binding accordingly).

The Last thing we need do is to enable the metadata creation in Sonarr and Radarr which is essential for ChineseSubFinder to work.
### Plex
I am using Plex but feel free to try Jellyfin and Emby for your own preference. I will mainly talk about how to get subtitles to work in Plex and there are two approaches for me:
- [Sub-Zero Subtitles](https://github.com/pannal/Sub-Zero.bundle) - which enables us to search the subtiles in opensubtitles.org
- [ChineseSubFinder](https://github.com/ChineseSubFinder/ChineseSubFinder) - which will detect the contents and auto download the subtitles

Sub-zero is working great if you need more than Chinese subtitles but I found it difficult to get matched for some new and less populare shows. Then I find ChinesSubFinder which is open-source project and work like a charm for me.

Both are easy to set up and there are a lot of tutorials just one google away. So I will not spend more time on that. Insetad I wanna point out the key for Plex to auto load the subtitles is using XBMCnfoTVImporter and XBMCnfoMoviesImporter as the proxy which could installed via WebTools (see [Git](https://github.com/gboudreau/XBMCnfoTVImporter.bundle)).

For example, in my Movies library, I set up the scanner as Plex Movie Scanner and the Proxy is XBMCnfoMoviesImporter as below:
![](https://i.imgur.com/YUAvBXf.png)

Hence we need refresh the library so these gears could work. 

## Conclusion
So far it is working very great to me and I don't have to search these subtitles anymore. Please leave comments if you have any questions.