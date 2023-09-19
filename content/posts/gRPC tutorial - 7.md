---
title: "gRPC tutorial - 7: Takeaway"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Programming"
tags: 
    - "gRPC"
    - "Microservices"
---

The four types of gRPC covers the majority use case when we design API under HTTP/2 since we could either send one request/response or multiple requests/responses in one call. 

And using proto buffer enables us to separate the implementation of service, server and client which are the three components we need develop for each RPC service. Though the service could also depend on several components. But the idea is the simple, we need implement service, server and client for each RPC service and we don't have to stick with one language for server and client.

The design of gRPC also simplifies the development that we just need implement the class in the `*grpc.class` and gRPC will do all heavy lifting for us. In this tutorial, the `BookService` is the subclass of ``
`BookServiceGrpc.BookServiceImplBase` which already contains 4 methods we need override including `createBook`, `uploadImage`, `recommendBook` and `rateBook`.
This exactly what we need implement in service on server side. For client side, what we need implement depends on the different type of stubs:
- Blocking Stub - RPC call will wait on a response or throw exception
- Async Stub - The response is returned asynchronously thus RPC call is not blocking.
- Future Stub - which is similar with Blocking Stub.

In our implementation, we choose async stub and there is another `asyncStub.rateBook` method we could use to call `rateBook` on server side. It might look odd, but they are actually the same `rateBook` method gRPC is going to invoke. The below is the method we implement in `BookService` and what we do it to return `StreamObserver<RateBookRequest>` and directly use `responseObserver` to send the response.
```java
public io.grpc.stub.StreamObserver<today.ihelio.grpc.RateBookRequest> rateBook(  
    io.grpc.stub.StreamObserver<today.ihelio.grpc.RateBookResponse> responseObserver) {  
  return io.grpc.stub.ServerCalls.asyncUnimplementedStreamingCall(getRateBookMethod(), responseObserver);  
}
```

In client, we call `asyncStub.rateBook` and pass in `StreamObserver<RateBookResponse>`. 
```java
StreamObserver<RateBookRequest> requestObserver = asyncStub  
        .rateBook(new StreamObserver<RateBookResponse>() {  
            @Override  
            public void onNext(RateBookResponse response) {  
                logger.info(String.format("laptop rated: id = %s, count = %s, avg = %s",  
                        response.getBookId(),  
                        response.getRatingCount(),  
                        response.getAvgRating()));  
            }  
  
            @Override  
            public void onError(Throwable t) {  
                logger.log(SEVERE, "rating failed: " + t.getMessage());  
                finishLatch.countDown();  
            }  
  
            @Override  
            public void onCompleted() {  
                logger.info("rate laptop completed");  
                finishLatch.countDown();  
            }  
        });
```

Bascially we implement `RequestObserver` and `ResponseObserver` on server and client side respectively but for the same `rateBook` method. gRPC helps us to use the same method on server and client without worrying about how they are wired up.

It is everything I want to cover in this gRPC tutorial - the implementation of different kinds of gRPC call and how gRPC simplifies our development.

I believe it will give you a good start if you are new to gRPC and please leave your comments or questions below. I'd love to discuss any topics with you.

Big thanks to this course [Tech School Videos](https://www.youtube.com/watch?v=2Sm_O75I7H0&list=PLy_6D98if3UJd5hxWNfAqKMr15HZqFnqf)
