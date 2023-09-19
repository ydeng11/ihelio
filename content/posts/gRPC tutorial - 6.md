---
title: "gRPC tutorial - 6: Bidirectional streaming"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Programming"
tags: 
    - "gRPC"
    - "Microservices"
---

We will speed it up a bit in this chapter after implementing unary call and one direction streaming. As always, we need implement our core logic which includes our book store which handles rating update and computing and book service which handls the request and return response. 

```java
@Override  
public Book rateBook(String bookID, Integer rating) {  
    Book book = inMemoryBookStore.getOrDefault(bookID, null);  
    if (book == null) {  
        throw NOT_FOUND.withDescription("book not found")  
                .asRuntimeException();  
    }  
    inMemoryBookStore.computeIfPresent(bookID, (k, v) -> {  
        Integer oldRating = v.getRating();  
        Integer oldCount = v.getRatingCount();  
        return v.toBuilder()  
                .setRating(rating + oldRating)  
                .setRatingCount(oldCount + 1)  
                .setAvgRating((rating + oldRating)/(float) (oldCount + 1))  
                .build();  
    });  
    return inMemoryBookStore.get(bookID);  
}
```

we simply update the rating in our book store.

We also need add `rateBook` method to `BookService`.

```java
@Override  
public StreamObserver<RateBookRequest> rateBook(StreamObserver<RateBookResponse> responseObserver) {  
    return new StreamObserver<RateBookRequest>() {  
        @Override  
        public void onNext(RateBookRequest request) {  
            String bookID = request.getBookId();  
            Integer rating = request.getRating();  
            Book book = bookStore.rateBook(bookID, rating);  
            RateBookResponse response = RateBookResponse.newBuilder()  
                    .setBookId(book.getId())  
                    .setRatingCount(book.getRatingCount())  
                    .setAvgRating(book.getAvgRating())  
                    .build();  
            responseObserver.onNext(response);  
        }  
  
        @Override  
        public void onError(Throwable t) {  
            logger.log(SEVERE, "rating failed " + t.getMessage());  
        }  
  
        @Override  
        public void onCompleted() {  
            logger.info("rating finished");  
        }  
    };  
}
```

As client streaming, we need return a `StreamObserver<RateBookRequest>` and fill the three Override methods.

The last we need update is client. Since it is a bidirectional streaming, we need pass in `StreamObserver<RateBookResponse>` to `asyncStub.rateBook` and get a 
`StreamObserver<RateBookRequest>` back. 

```java
public void rateBook(String[] bookIDs, Integer[] ratings) throws InterruptedException {  
    CountDownLatch finishLatch = new CountDownLatch(1);  
    logger.info("rating started");  
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
  
    int n = bookIDs.length;  
    try {  
        for (int i = 0; i < n; i++) {  
            RateBookRequest request = RateBookRequest.newBuilder()  
                    .setBookId(bookIDs[i])  
                    .setRating(ratings[i])  
                    .build();  
            requestObserver.onNext(request);  
            logger.info("sent rate-book request: id = " + request.getBookId() + ", score = " + request.getRating());  
        }  
    } catch (Exception e) {  
        logger.log(Level.SEVERE, "unexpected error: " + e.getMessage());  
        requestObserver.onError(e);  
        return;  
    }  
  
    requestObserver.onCompleted();  
    if (!finishLatch.await(1, TimeUnit.MINUTES)) {  
        logger.warning("request cannot finish within 1 minute");  
    }  
  
}
```

For `responseObserver`, we just log the response data. The returned requestObserer is the one we defined in `BookService`. Then the requestObserver will handle the requests sequentially until none. In the end, we should call `onCompleted` which will just simply log the message.

Now we have implemented four kinds of gRPC method. I will wrap up everything we learned in this tutorial next chapter. 

The complete project can be found [here](https://github.com/ydeng11/gRPC-tutorial/tree/main).