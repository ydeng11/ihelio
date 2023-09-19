---
title: "Why I don't like WSL"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - Programming
tags: 
    - WSL
    - Environment

---

**If you are not using Java or Intellij Idea for your project, you could stop reading.**

Couple months ago, I am very excited with WSL2 and it works perfectly for me to work on some Java projects in Intellij Idea. Somehow everything is upside down and obviously after some updates of windows though I still stick with win10.

Long story short! **Intellij Idea is really slow with WSL regarding the version**. The indexing takes forever and it is not new as I found in this [thread](https://youtrack.jetbrains.com/issue/WI-63786/Working-with-projects-on-WSL-is-extremely-slow-basically-not-possible-to-work-with) and this [thread](https://youtrack.jetbrains.com/issue/IDEA-274193/IntelliJ-extremely-slow-on-listing-directory-contents-of-WSL-filesystem) and the problem could be one of the following:
1. Intellij is accesing the files through the mount which is not ideal;
2. Windows security is doing virus scan which could also impact the speed
Given the suggestions in these threads, I exclude a bunch of files and folders including the idea.exe and `\\wsl$\Ubuntu\home\user\project` and none of them work for me and obviously other people.

Then I tried VS code and it obviously has another problem with adding `target/generated-sources`  to the class path in maven. I spent hours on it and I decided to quit as I really prefer Idea than VS code from Java perspective.

So I spend another several hours installing Ubuntu as a double system for me to code Java specfically. I think WSL is diffiuclt with Intellij now since I saw many other complains with other Intellij softwares and the supporter of Intellij said there is nothing they could do since the problem is induced by the change from windows. 

My advice to people in my situation - a PC gamer and need code more than Python - embrace the idea of double system and use cloud drive to sychornize.  