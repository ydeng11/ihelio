---
title: "gRPC tutorial - 4: Client Streaming"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Programming"
tags: 
    - "gRPC"
    - "Microservices"
---

We will implement a function to upload image when we create the books like the cover and something else. And we would split the image into chunks and we upload them chunk by chunk until all data are transfered. As we defined earlier in the proto, we could have multiple images for one book. So we will need a function called `uploadImage` and uploade all images one by one.

Let's start with the proto of image:

```proto
syntax = "proto3";  
package book;  
option java_multiple_files = true;  
option java_package = "today.ihelio.grpc";  
  
import "google/protobuf/timestamp.proto";  
  
message Image {  
  string id = 1;  
  string file_path = 2;  
  string size = 3;  
  google.protobuf.Timestamp uploaded_at = 4;  
}
```

We create an `Image` message which has four fields: image id, file path in local, size and timestamp. This will be used on server side to contain the image info when it receives an image from client.

Then we need define our request and response for client.

```proto
message UploadImageRequest {  
  oneof data {  
    ImageInfo info = 1;  
    bytes chunk_data = 2;  
  }  
}  
  
message ImageInfo {  
  string book_id = 1;  
  string image_type = 2;  
  string file_name = 3;  
}  
  
message UploadImageResponse {  
  string id = 1;  
  uint32 size = 2;  
  string file_path = 3;  
}
```

We define an `ImageInfo` message which will contain book_id it associates with, image type and name. Thus the `UploadImageRequest` would upload the info to the server in addition to the image itself. Then the `UploadImageResponse` would return the server-generated id, size and file path at server.

Thus the `BookService` now is:
```proto
service BookService {  
  rpc CreateBook(CreateBookRequest) returns (CreateBookResponse) {};  
  rpc UploadImage(stream UploadImageRequest) returns (UploadImageResponse) {};
  }

```

Now we should turn to the implmentation of `UploadImage` in server, client and service. Since we already have them, we will now build upon them.

`BookService` always comes the first since it is our core logic. With core logic built up, it is easy to implement server and client for it. And it is easy to add new feature to `BookService` given we just need use the auto code generation function in Intellij and insert the `Override Method` which is `uploadImage` in this case.

```java
@Override  
    public StreamObserver<UploadImageRequest> uploadImage(StreamObserver<UploadImageResponse> responseObserver) {  
        return new StreamObserver<UploadImageRequest>() {  
            private String bookID;  
            private ImageInfo info;  
            private ByteArrayOutputStream imageData;  
  
            @Override  
            public void onNext(UploadImageRequest request) {  
                if (request.getDataCase() == UploadImageRequest.DataCase.INFO) {  
                    info = request.getInfo();  
                    logger.info("receive image info:\n" + info);  
  
                    bookID = info.getBookId();  
                    imageData = new ByteArrayOutputStream();  
  
                    Book found = bookStore.findBook(bookID);  
                    if (found == null) {  
                        responseObserver.onError(  
                                Status.NOT_FOUND  
                                        .withDescription("Book not found")  
                                        .asRuntimeException()  
                        );  
                    }  
                    return;  
                }  
  
                ByteString chunkData = request.getChunkData();   
  
                if (imageData == null) {  
                    logger.info("image info wasn't sent before");  
                    responseObserver.onError(  
                            Status.INVALID_ARGUMENT  
                                    .withDescription("image info wasn't sent before")  
                                    .asRuntimeException()  
                    );  
                    return;  
                }  
  
                try {  
                    chunkData.writeTo(imageData);  
                } catch (IOException e) {  
                    responseObserver.onError(  
                            Status.INTERNAL  
                            .withDescription("cannot write chunk data: " + e.getMessage())  
                                    .asRuntimeException());  
                }  
            }  
  
            @Override  
            public void onError(Throwable t) {  
                logger.warning(t.getMessage());  
            }  
  
            @Override  
            public void onCompleted() {  
                String imageID = "";  
                int imageSize = imageData.size();  
                try {  
                    imageID = imageStore.Save(bookID, info, imageData);  
                } catch (IOException e) {  
                    throw new RuntimeException(e);  
                }  
                UploadImageResponse response = UploadImageResponse.newBuilder()  
                        .setId(imageID)  
                        .setSize(imageSize)  
                        .build();  
                responseObserver.onNext(response);  
                responseObserver.onCompleted();  
            }  
        };  
    }
```

Same as alwasys, the key is to return what we need return which is `StreamObserver<UploadImageRequest>`, thus we could directly return a new `StreamObserver<UploadImageRequest>` and put our logic inside the code block.

And there are three function we need fill:
- OnNext: continously handle the next request
	- Create `ByteArrayOutputStream` to save the image data
	- Check if the upload data type is `ImageInfo`
	- If yes, we store the image info, otherwise we write the data chunk to `ByteArrayOutputStream`
- OnError: 
	- Handle the error
- OnCompleted:
	- Save the image to our image store (we will talk about it later)
	- Send out the response
	- Close the response

After we have the logic ready, we then can start client and server implementation.

The following snippet is the implmenetation of `uploadImage` from client:
```java
public void uploadImage(String bookID, Image image) throws InterruptedException {  
    final CountDownLatch latch = new CountDownLatch(1);  
  // we use asyncStub instead of blockingStub for streaming connection
    StreamObserver<UploadImageRequest> requestObserver = asyncStub.withDeadlineAfter(5, SECONDS)  
            .uploadImage(new StreamObserver<UploadImageResponse>() {  
                @Override  
                public void onNext(UploadImageResponse response) {  
                    logger.info("receive response:\n" + response);  
                }  
  
                @Override  
                public void onError(Throwable t) {  
                    logger.log(SEVERE, "upload failed: " + t);  
                    latch.countDown();  
                }  
  
                @Override  
                public void onCompleted() {  
                    logger.info("image uploaded");  
                    latch.countDown();  
                }  
            });  
    FileInputStream fileInputStream;  
    String imagePath;  
    try {  
        imagePath = image.getFilePath();  
        fileInputStream = new FileInputStream(imagePath);  
    } catch (FileNotFoundException e) {  
        logger.log(SEVERE, "cannot read image file: " + e.getMessage());  
        return;  
    }  
  
    String imageType = imagePath.substring(imagePath.lastIndexOf("."));  
    String fileName = imagePath.substring(imagePath.lastIndexOf("/"));  
    ImageInfo info = ImageInfo.newBuilder()  
            .setBookId(bookID)  
            .setImageType(imageType)  
            .setFileName(fileName)  
            .build();  
    UploadImageRequest request = UploadImageRequest.newBuilder().setInfo(info).build();  
  
    try {  
        requestObserver.onNext(request);  
        logger.info("send image info:\n" + info);  
  
        byte[] buffer = new byte[1024];  
        while (true) {  
            int n = fileInputStream.read(buffer);  
            if (n <= 0) {  
                break;  
            }  
  
            if (latch.getCount() == 0) {  
                return;  
            }  
            request = UploadImageRequest.newBuilder()  
                    .setChunkData(ByteString.copyFrom(buffer))  
                    .build();  
            requestObserver.onNext(request);  
        }  
    } catch (Exception e) {  
            logger.log(SEVERE, "unexpected error: " + e.getMessage());  
        }  
    requestObserver.onCompleted();  
  
    if (!latch.await(1, TimeUnit.MINUTES)) {  
        logger.warning("request cannot finish within 1 minute");  
    }  
}
```

Though it looks confused in the beginning since we have `response` as the parameter of `uploadImage` and get a `request` back which makes it conter-intuitive compared with `createBook`, it would help you to get it straight if you think about the meaning of streaming. 

We are streaming the request to the server, which means we have multiple requests and we need something to handle them sequentially. Thus we need something might sound like request handler - which is exactly `StreamObserver<UploadImageRequest>`. In other words, `uploadImage` yield a request handler instead of one time response as `createBook`.

With this request handler, we just need pass in request till none. Therefore, we create `StreamObserver<UploadImageRequest>` by passing in a simple `StreamObserver<UploadImageResponse>` which also contains OnNext, OnError and OnCompleted (we will need fill the three methods as long as it is streaming connection).

Since it is the response, we could just simply log the info. Next step, we should starting building these requests. We will build `UploadImageRequest` from either `ImageInfo` (from `Image` object) or `ChunkData` (from `fileInputStream`), and send them sequentially to the request handler. 

In the end, we should just call `onCompleted` to shut down the streaming.

Accordingly, we need revise `createBook` in client to upload images when creating book:

```java
public void createBook(Book book) throws InterruptedException {  
    CreateBookRequest request = CreateBookRequest.newBuilder().setBook(book).build();  
    CreateBookResponse response = CreateBookResponse.getDefaultInstance();  
  
    try {  
        response = blockingStub.withDeadlineAfter(5, SECONDS).createBook(request);  
  
        for (Image image : book.getImageList()) {  
            new Thread(  
                () -> {  
                    try {  
                        uploadImage(book.getId(), image);  
                    } catch (InterruptedException e) {  
                        throw new RuntimeException(e);  
                    }  
                }  
                ).start();  
        }  
  
    } catch (StatusRuntimeException e) {  
        if (e.getStatus().getCode() == Status.Code.ALREADY_EXISTS) {  
            logger.info("Book already exists");  
            return;  
        }  
    } catch (Exception e) {  
        logger.log(SEVERE, "request failed " + e.getMessage());  
    }  
    logger.info("New book created: \n" + response.getId());  
}
```

For each image, we create a new thread to upload the image. **The standard way is to use a Executor Pool instead of starting a new thread whatever.** But bear with me since the meat is gRPC here.

It is a lot to take from serverice and client. Now it is the easist part, server. But before that, we prob need implement image store for us since we would need it in our server implementation. Same as book store, we will implement the image store in disk. To be clear, the image itself is saved in local like a blob store and the path will be saved in memory. So we can easily access the image for a book from the book store.

```java
package today.ihelio.learngrpc;  
  
import today.ihelio.grpc.Image;  
import today.ihelio.grpc.ImageInfo;  
  
import java.io.ByteArrayOutputStream;  
import java.io.FileNotFoundException;  
import java.io.FileOutputStream;  
import java.io.IOException;  
import java.util.UUID;  
import java.util.concurrent.ConcurrentHashMap;  
import java.util.concurrent.ConcurrentMap;  
  
import static com.google.protobuf.util.Timestamps.fromMillis;  
import static java.lang.System.currentTimeMillis;  
  
/**  
 * @author helio  
 * @date 2022/9/4  
 * @package today.ihelio.learngrpc  
 */public class DiskImageStore implements ImageStore{  
  
    ConcurrentMap<String, ConcurrentMap<String,Image>> store;  
    private String imageFolder;  
  
    public DiskImageStore(String imageFolder) {  
        this.store = new ConcurrentHashMap<>();  
        this.imageFolder = imageFolder;  
    }  
  
    @Override  
    public String Save(String bookID, ImageInfo info, ByteArrayOutputStream imageData) throws IOException {  
        String imageName = info.getFileName();  
        String imageID = UUID.nameUUIDFromBytes(imageName.getBytes()).toString();  
        if (store.containsKey(bookID)) {  
            if (store.get(bookID).containsKey(imageID)) {  
                throw new AlreadyExistsException("image %s already existed for book %s".formatted(imageName, bookID));  
            }  
        } else {  
            store.put(bookID, new ConcurrentHashMap<>());  
        }  
  
        String imagePath = String.format("%s/%s", imageFolder, imageName);  
  
        FileOutputStream fileOutputStream = new FileOutputStream(imagePath);  
        imageData.writeTo(fileOutputStream);  
        fileOutputStream.close();  
  
        Image image = Image.newBuilder()  
                .setId(imageID)  
                .setFilePath(imagePath)  
                .setUploadedAt(fromMillis(currentTimeMillis()))  
                .build();  
  
        store.get(bookID).put(imageID, image);  
  
        return image.getId();  
    }  
}
```

The `DiskImageStore` is simple, since we are passing `ByteArrayOutputStream`, we just need create the image path and save the image to the path, and update the `InMemoryStore`. 

For server, we would need revise it a bit since `BookService` requires `DiskImageStore` now but the most remain the same.

```java
package today.ihelio.learngrpc;  
  
import io.grpc.Server;  
import io.grpc.ServerBuilder;  
import io.grpc.protobuf.services.ProtoReflectionService;  
  
import java.io.IOException;  
import java.util.concurrent.TimeUnit;  
import java.util.logging.Logger;  
  
/**  
 * @author helio  
 * @date 2022/9/3  
 * @package today.ihelio.learngrpc  
 */public class BookServer {  
    private final Logger logger = Logger.getLogger(BookServer.class.getName());  
    private BookService bookService;  
    private final int port;  
    private final Server server;  
  
    public BookServer(int port, BookStore bookStore, ImageStore imageStore) {  
        this(ServerBuilder.forPort(port), port, bookStore, imageStore);  
    }  
  
    public BookServer(ServerBuilder serverBuilder, int port, BookStore bookStore, ImageStore imageStore) {  
        this.port = port;  
        this.bookService = new BookService(bookStore, imageStore);  
        this.server = serverBuilder.addService(bookService)  
                .addService(ProtoReflectionService.newInstance())  
                .build();  
    }  
  
    public void start() throws IOException {  
        server.start();  
        logger.info("Book server started on port " + port);  
  
        Runtime.getRuntime().addShutdownHook(new Thread(){  
            @Override  
            public void run() {  
                System.err.println("shutdown gRPC server because JVM shuts down");  
                try {  
                    BookServer.this.stop();  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
                System.err.println("server shutdown");  
            }  
        });  
    }  
  
    public void stop() throws InterruptedException{  
        if (server != null) {  
            server.shutdown().awaitTermination(30, TimeUnit.SECONDS);  
        }  
    }  
  
    private void blockUntilShutdown() throws InterruptedException {  
        if (server != null) {  
            server.awaitTermination();  
        }  
    }  
  
    public static void main(String[] args) throws IOException, InterruptedException{  
        InMemoryBookStore inMemoryBookStore = new InMemoryBookStore();  
        DiskImageStore diskImageStore = new DiskImageStore("src/main/resources/images_destination");  
        BookServer server = new BookServer(9080, inMemoryBookStore, diskImageStore);  
        server.start();  
        server.blockUntilShutdown();  
    }  
  
}
```

Let's wrap up the client streaming in gRPC:
1. We need update proto to include necessay message and service
2. Implement the core logic in the service, client and other revisions needed
	1. What we implement is a request handler, not a single request-response function

Next chaper will introduce server streaming connection which has a lot in common with client streaming.

The complete project can be found [here](https://github.com/ydeng11/gRPC-tutorial/tree/main).

