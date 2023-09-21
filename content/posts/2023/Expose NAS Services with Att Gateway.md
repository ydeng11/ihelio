---
title: "Expose NAS Services with Att Gateway"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Home Server"
tags: 
    - "NAS"
---

Recently I changed my ISP to ATT to try their fiber, and they gave me the bgw320 as the gateway for the Internet service. And I have trouble connecting to my Synology's services like Jellyfin. 

I suspect it has some conflicts with the network of these docker services running in Synology or the internet setup of Synology. Considering the configuration of docker network could be another rabbit hole, I didn't go that route. I decided to connect the gateway with my own router and use the passthrough function in the gateway so I can do the port forwarding in my router. 

To make it work, I also need to reset the internet connection in my Synology which somehow is not accessible after I made this change. Since I am not very good at network, I don't know what I did wrong,  but I am glad reset just made it work.

To wrap up what I did to make it work:
-   Connect a router to the bgw320 as passthrough 
![](https://i.imgur.com/Jq3PDNO.png)
- [Reset the network setting](https://goabacus.com/how-to-reset-synology-nas-three-ways) 
	- In my case, my subnet changes to 10.0.0.0 from 192.168.0.0 which might be the reason I have to reset the network in Synology
- All the docker services are running at the new IP with the same port afterwards