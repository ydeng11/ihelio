---
title: "Sneak peek at the asynchronous Java"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - Programming
tags: 
    - JAVA
    - Asynchronous
    - Loom Project
---


Java 19 is here with the preview of loom project. Loom project is the one helps Java become asynchronous and come back to the table to compete with other asynchronous language such as Golang. But why do we need Java to be asynchronous?

## Blocking
It is all because we don't want to be blocked. And the best example must be how you do your driver license renewal at the DMV. I remember I work up at 5:30 in the morning and drove to the DMV where there was already a line of 50 people. But I have no choice but to join them. After several hours' waiting, I can finally check in and get a number for my case. Then I was eligible to walk in and find a chair to continue my waiting. After another couple hours, I finally got called at a window and finished my license renewal task.

It is very obvious I am blocked before and after the check-in. How should we make this more efficient that no one should be blocked? The simple answer is to increase the efficiency and the number of staff at the DMV. But we both know it is impossible to have 100 staff at one single DMV to deal with all the services. 

## Concurrency but no Asychronous
This is exactly what we are facing with in Java. Multi-threading is important for us to reduce the blocking in Java, but the number of threads is restricted by the machine. Moreover, thread is usually tied to a specific task which is like the staff of the DMV is assigned to one customer at a time and cannot help other customers unless they finished their current work. Thus, Java can achieve concurrency but can never reach asynchronous with current threading mechanism. 

## VirtualThread
This why we need the Loom project which provides `VirtualThread` which is super lightweight because it is now managed by JVM instead of OS thus the number of `VirtualThread` could go way beyond the number of cores. In association with the new data structure, `continuation`, Loom also enables these `VirtualThread` to move around, if their current task is blocked like working on some IO task, and very likely another `VirtualThread` could come to pick up what is left and finish the rest task.

To show the power of Loom, I am going to use a very simple snippet. 
**Reminder: We need to change the language level to enable preview for Java 19 if we want to use it.**
![](https://i.imgur.com/BOhqmVs.png)

In the code below, we can tweak the `numerThreads` to test how many threads we could create with normal `Thread` and `VirtualThread`. For each thread, we assign a simple task which just sleep for 500 ms. In my test, I can create 1,000 threads but failed with 5,000 threads. When we broke the limit, we would get an `OutOfMemeryError`. To the opposite, I can create even 10,000 `VirtualThread` which means I could handle more request at the same time and blocking will be largely reduced.

```java
public class Main {  
  public static void doSomething(){  
    try {  
      Thread.sleep(500);  
    } catch (InterruptedException e) {  
      throw new RuntimeException(e);  
    }  
  }  
  
  public static void main(String[] args) throws InterruptedException {  
    int numberThreads = 10_000;  
    Thread thread = null;  
    // Normal Thread  
    //for (int i = 0; i < numberThreads; i++) {    //  thread = new Thread(Main::doSomething);    //  thread.start();    //}    //    //thread.join();  
    // Virtual Thread    for (int i = 0; i < numberThreads; i++) {  
      thread = Thread.startVirtualThread(Main::doSomething);  
    }  
    thread.join();  
  }  
}
```
It is a very exciting achievement and feature to Java community. I believe it would bring a lot of changes to Java standards as well.

---
# References
[# What's Looming in Java The Why and What of Project Loom - Venkat Subramaniam](https://youtu.be/y-SXxp1Kx_Y)