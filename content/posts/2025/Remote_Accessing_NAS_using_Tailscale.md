---
title: "Remote Accessing NAS using Tailscale"
date: 2025-01-25T22:42:13-05:00
draft: false
categories: 
    - Selfhosted
tags: 
    - Tailscale
    - VPN
---

# Remote Accessing NAS using Tailscale

Simply put, Tailscale is a private VPN built on the WireGuard protocol, designed to support a Zero Trust architecture for managing devices within a subnet. As a mesh network, Tailscale enhances connectivity when accessing NAS services remotely. Compared to traditional VPNs and firewalls, WireGuard and Zero Trust offer significantly better security since every device requires authentication to communicate with others. This granular access control minimizes risk—even if an attacker compromises one device, they won't automatically gain access to the entire private network. Unlike conventional firewalls, which often have a hardened exterior but a vulnerable core, Tailscale ensures both outer and inner security.

### Methods to Access NAS Using Tailscale

#### 1. [Install Tailscale Directly on the NAS](https://tailscale.com/kb/1307/nas)

- This method turns the NAS into a node on the tailnet (your private network), making it discoverable by all other nodes.

- Once installed, Tailscale detects available services on the NAS, allowing easy access using `tailscale.domain:port` or `100.x.x.x:port`.

- **Pros:**
  
  - Extremely simple setup.
  - Fine-grained access control for devices and users.

- **Cons:**
  
  - Exposing all services makes me itchy but I don't think it is a con as we can have ACL to limit the access for users.
    
    #### 2. [Configure a Subnet Router](https://tailscale.com/kb/1019/subnets)

- A subnet router bridges your private LAN with the tailnet, making local network resources accessible remotely.

- Even unconventional devices, like an Apple TV, can act as a subnet router, which adds flexibility.

- Once configured, NAS services become accessible using `nas-ip:port` over the tailnet.

- **Pros:**
  
  - Easy to set up, especially on always-on devices like Apple TV or a Raspberry Pi.
  - Provides network-wide access without installing Tailscale on every device especially for those devices cannot install Tailscale like printer.

- **Cons:**
  
  - Exposes the entire LAN, which could require additional security measures such as reverse proxying - I didn't try if we can put ACL on the particular device in the subnet.
    
    #### 3. [Use Tailscale as a Sidecar for Docker Containers](https://tailscale.com/kb/1282/docker)

- Sidecar means we need couple Tailscale container with whatever services we want to remote access.

- By sharing Tailscale's network with another service (e.g., a web app or reverse proxy), the service seamlessly joins the tailnet with the help of Tailscale.

- If combined with a reverse proxy, this method can also facilitate access to LAN resources securely.

- **Pros:**
  
  - Provides fine-grained control over which services are remotely accessible so we don't have expose a bunch of services we don't want to remote access.

- **Cons:**
  
  - Requires more configuration and consumes additional resources.

Tailscale significantly improves security and usability for self-hosted services. However, another promising project worth mentioning is [Pangolin](https://github.com/fosrl/pangolin), *a **self-hosted tunneled reverse proxy management server with built-in identity and access management (IAM)***. Though still in beta at the time of writing, Pangolin offers exciting potential, particularly with its IAM management. I am very excited to try it is officially released.
