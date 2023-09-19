---
title: "gRPC tutorial - 1: Overview"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Programming"
tags: 
    - "gRPC"
    - "Microservices"
---

# What is gRPC

gRPC is a high performance open-source freature-rich RPC (remote procedure calls) framework developed by google. “g” stands for many different meaning like green, good and etc.

It is a protocal that allows a program to

-   execute a procedure of another program located in other computer
-   without the developer explicitly coding the details for the remote interaction

gRPC is one kind of s2s call and widely used to replace REST API in the backend due to:

- less usage of bandwidth since all data are serialized
- support different languages
- easy to focus on the core logic and gRPC will handle all the boiler-plate code for us

gRPC is defining the next-gen of microservices structure with http2. In other words, gRPC is providing the same functions of HTTP2:
![](https://i.imgur.com/jRNiFEm.png)

Thus gRPC outperforms REST in many cases:
![](https://i.imgur.com/kJypeVE.png)

# How gRPC works
-   client has a generated stub that provides the same methods as the server
-   the stub calls gRPC framework under the hood to exchange information over the network
-   since client and server use stubs to interact with each other, so they only need to implement their core service logic
![](https://i.imgur.com/tis58Zo.png)

# How Stubs work
gRPC will be transferred between server and client via Stubs. Stubs are generated via proto buffers and then we could implement the core logic through the stubs. This we need the proto buffer compiler (aka. protoc) and gRPC plugin for corresponding langague to generate the boiler-plate.

Using proto buffer provides:
1. human readable interface
2. strongly typed contract
3. efficient data serialization
4. one size fits all - all other languages can be generated from the same proto
5. compatibility

# Types of gRPC
![](https://i.imgur.com/LI8x5aC.png)

gRPC now is support 4 kinds of communication between server and client:
1. unary - like REST call, each request will return an response.
2. streaming
	1. client streaming - client is sending multiple requests within one call
	3. server streaming - sever is sending multiple responses within one call
	4. bidrectional streaming - server and client are sending multiple requests and responses to each other within one call

**We will build a simple book service to implement the 4 kinds of gRPC:**
1. Unary -> CreateBook Service
2. ClientStreaming -> UploadImage Service
3. ServerStreaming -> RecommendBook Service
4. Bidirectional Streaming -> RateBook Service
