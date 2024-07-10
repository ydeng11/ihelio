---
title: "How I Upgraded My Mac Mini Storage with an External SSD and Solved the Home Directory Issue"
date: 2024-07-09T21:37:51-04:00
draft: false
categories: 
    - System
tags: 
    - Mac mini
    - External Drive
    - MacOS
---

A couple of months ago, I had the brilliant idea to upgrade my Mac mini M2 by installing MacOS on an external M.2 SSD. For under 200 bucks, I went from a measly 250GB to a whopping 2TB. I was on cloud nine, reveling in my newfound digital real estate, until the other weekend when the system decided to crash my party by refusing to download upgrades due to a lack of space. Wait, what?

Turns out, despite my genius plan, the OS on the external drive didn’t update the home directory. So, all my apps were still partying it up on the internal drive, leaving it packed to the brim. Cue the dramatic realization that my internal storage was maxed out.

First things first, I tried clearing out my Documents and Downloads folders. But it was like bailing water from the Titanic with a teaspoon—useless. The solution? Change the home directory. Simple, right? You’d think.

Here’s where you go to do it: `Users & Groups`.

![image1](app://0e68ac1fe83e5c79e82d0b7f61f92bb563bb/Users/ihelio/Documents/Zettelkasten/Pasted%20image%2020240708204503.png?1720485903633)

![image2](app://0e68ac1fe83e5c79e82d0b7f61f92bb563bb/Users/ihelio/Documents/Zettelkasten/Pasted%20image%2020240708204535.png?1720485935887)

Easy peasy. Except, my Mac threw a tantrum and refused to restart, getting stuck on the loading screen. After some frantic Googling, I discovered this is a common issue when changing the home directory. Brilliant. I managed to log in via safe mode, and somehow the home directory was updated, but the regular mode was still broken.

At this point, I decided to go nuclear and reinstall the system. My theory? The problem started because I transferred my system from the internal drive to the external drive without updating the home directory. So, I erased everything and reinstalled MacOS on both the internal and external drives separately. Finally, I had different home directories, and the storage information was correct. Success!

![](https://i.imgur.com/nhdRArq.png)

Looking back, I think the internal drive was still accessible by the system on the external drive, which is why it "worked" even with the messed-up home directory. I watched tons of videos about installing MacOS on external drives, but none of them mentioned this hiccup. Maybe it’s a rare case, but here’s my advice: if you’re transferring your system to an external drive, make sure to check and update your home directory.

Lessons learned? Always double-check your home directory, expect the unexpected, and keep your sense of humor handy. You never know when your tech setup will turn into a comedy of errors.