---
title: "gRPC tutorial - 5: Server Streaming"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Programming"
tags: 
    - "gRPC"
    - "Microservices"
---

Regarding server streaming, we will implement a simplified recommendation service to recommend the books based on the popularity. Like client streaming, we would need a response handler - `StreamObserver<RecommendBookResponse>` - to handle a sequential of responses given one request.

Since this is a recommendation function, we would need search the books from our book store given the criteria. Thus we would need add a `searchBook` method to our book store. 

```java
@Override  
public void searchBook(Context ctx, Integer popularity, BookStream bookStream) {  
    for (Map.Entry<String, Book> entry : inMemoryBookStore.entrySet()) {  
        if (ctx.isCancelled()) {  
            logger.info("context is cancelled");  
            return;  
        }  
  
        Book book = entry.getValue();  
        if (book.getPopularity() >= popularity) {  
            bookStream.Send(book.getId());  
        }  
    }  
}
```

The `searchBook` need three vairables: 
- context: check if connection is still available
- popularity: used to find the recommended books
- bookStream: the wrapper of `StreamObserver<RecommendBookResponse>` to send back the responses (bookStream is an interface and we will implement a lambda function for it in `BookService`).

Then we can add `recommenBook` method to `BookService` to handle the request and keep sending back the responses if any.

```java
@Override  
public void recommendBook(RecommendBookRequest request, StreamObserver<RecommendBookResponse> responseObserver) {  
    Integer popularity = request.getPopularity();  
  
    logger.info("checked all books with popularity greater than " + popularity);  
    bookStore.searchBook(Context.current(), popularity, new BookStream() {  
        @Override  
        public void send(String bookID) {  
            logger.info("found book: " + bookID);  
            RecommendBookResponse response = RecommendBookResponse.newBuilder().setBookId(bookID).build();  
            responseObserver.onNext(response);  
        }  
    });  
    responseObserver.onCompleted();  
    logger.info("finished book recommendation!");  
  
}
```

As we have gone through in client streaming, we could just auto insert the override method and we would know what variables we could use: request and responseObserver. Request is easy to understand and responseObserver is just the response handler. Noticebly there is nothing to return here because response are all handled inside this method by the response handler - ``StreamObserver<RecommendBookResponse>`.

We firstly get the popularity from our request - this could be way more complicated in real-world recommendation system - and we can feed this to `bookStore` to search the qualified books. And the last variable is `bookStream` we metioned earlier. 

`bookStream` is essentially the wrapper of `StreamObserver<RecommendBookResponse>` and we could directly pass in `StreamObserver<RecommendBookResponse>` if we want. In `bookStream`, we implement `send` method to send out the response which just call `StreamObserver<RecommendBookResponse>` to pass in response.

Then we need shutdown the connection after finishing one recommendation service.

You might wonder where we implement `responseObserver` and how `onNext` is working. Well, we don't need in this case since we are using `blockingStub` which only requires `request` as the input. However, we would need `responseObserver` if we are using async stub. Thus, we need think about where we would call `recommendBook`. The answer is our client and we should create `responseObserver` in our client for it to handle response in async way (we will discuss the differences between blockingStub and asyncStub when we wrap up).

```java
public void recommendBook(Integer popularity) {  
    logger.info("search started");  
    RecommendBookRequest request = RecommendBookRequest.newBuilder()  
            .setPopularity(rand.nextInt(100))  
            .build();  
  
    try {  
        Iterator<RecommendBookResponse> iterator = blockingStub  
                .withDeadlineAfter(5, SECONDS)  
                .recommendBook(request);  
  
        while (iterator.hasNext()) {  
            RecommendBookResponse response = iterator.next();  
            logger.info("found: " + response.getBookId());  
        }  
    } catch (Exception e) {  
        logger.log(SEVERE, "request failed: " + e.getMessage());  
        return;  
    }  
    logger.info("recommendation completed!");  
  
}
```

Here we would get an iterator of `RecommendBookResponse` by calling `recommendBook` and we could do futher process like refine the ranking and do another filtering. Now we just log everything we received.

For server, there is nothing we need do now since server only need call the service when receiving the request. Thus we don't need change it as long as our service signature remains the same which makes saves a lot of time for us to focus on the core logic.

Next chapter will be last one - bidirectional streaming - and we will implement a rating function for client to submit ratings for several books at the same time.

The complete project can be found [here](https://github.com/ydeng11/gRPC-tutorial/tree/main).