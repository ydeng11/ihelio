---
title: "gRPC tutorial - 2: Enviroment Setup"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Programming"
tags: 
    - "gRPC"
    - "Microservices"
---

# Environment Setup

I am using Gradle and Intellij Idea for this project (Maven setting can be found [here](https://vsbogd.github.io/coding/using-grpc-in-java-maven-project.html)). The Gradle setting is shown as following:
```java
plugins {  
    id "com.google.protobuf" version "0.8.18"  
    id "java"  
}  
  
group 'today.ihelio.grpc.tutorial'  
version '1.0-SNAPSHOT'  
  
sourceCompatibility = 15  
  
repositories {  
    mavenCentral()  
}  
  
dependencies {  
    implementation 'junit:junit:4.13.1'  
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.1'  
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.1'  
    implementation group: 'com.google.protobuf', name: 'protobuf-java', version: '3.21.4'  
    runtimeOnly group: 'com.google.protobuf', name: 'protobuf-java-util', version: '3.21.4'  
    implementation group: 'io.grpc', name: 'grpc-all', version: '1.45.1'  
    runtimeOnly group: 'io.grpc', name: 'grpc-services', version: '1.48.0'  
    implementation group: 'javax.annotation', name: 'javax.annotation-api', version: '1.3.2'  
}  
  
test {  
    useJUnitPlatform()  
}  
  
sourceSets {  
    main {  
        java {  
            srcDirs 'build/generated/source/proto/main/grpc'  
            srcDirs 'build/generated/source/proto/main/java'  
            srcDirs 'src/main/resources'  
        }  
    }  
}  
  
protobuf {  
    protoc {  
        artifact = 'com.google.protobuf:protoc:3.21.4'  
    }  
  
    plugins {        grpc {  
            artifact = 'io.grpc:protoc-gen-grpc-java:1.49.0'  
        }  
    }  
  
    generateProtoTasks {  
        all()*.plugins {  
            grpc {}  
        }  
    }  
}  
  
jar {  
    from {  
        configurations.runtimeClasspath.findAll {  
            duplicatesStrategy = DuplicatesStrategy.EXCLUDE  
            it.name.endsWith(".jar")  
        }.collect {  
            println 'add ' + it.name  
            zipTree(it)  
        }  
    }  
}  
targetCompatibility = JavaVersion.VERSION_15
```
After we have the proto files, then we could just build the project to complie proto files and generate essential stubs.

And to test the service we created, we need a helper function to generate books for us.

```java
package today.ihelio.sample;  
  
import today.ihelio.grpc.Book;  
import today.ihelio.grpc.Image;  
import today.ihelio.grpc.Sample;  
  
import java.util.*;  
  
import static com.google.protobuf.util.Timestamps.fromMillis;  
import static java.lang.System.currentTimeMillis;  
  
/**  
 * @author helio  
 * @date 2022/9/3  
 * @package today.ihelio.sample  
 */public class Generator {  
    String[] genreList = new String[]{  
        "FICTION",  
        "MYSTERY",  
        "THRILLER",  
        "HORROR",  
        "HISTORICAL",  
        "ROMANCE",  
        "SCI_FICTION",  
    };    private final Random rand;  
    public Generator() {  
        rand = new Random();  
    }  
    private String generateRandomWords(int lengthofWords)  
    {        Random rand = new Random();  
        StringBuilder sb = new StringBuilder();  
        Random random = new Random();  
            // words of length 3 through 10. (1 and 2 letter words are boring.)  
        for(int j = 0; j < lengthofWords; j++)  
        {            sb.append((char)('a' + random.nextInt(26)));  
            if (rand.nextInt(10) == 1) {  
                sb.append(" ");  
            }        }        return sb.toString();  
    }    public String randomAuthor() {  
        return generateRandomWords(10);  
    }  
    public String randomName() {  
        return generateRandomWords(20);  
    }  
    public String randomPublication() {  
        return generateRandomWords(8);  
    }  
    public Sample randomSample() {  
        return Sample.newBuilder()  
                .setParagraph(  
                        generateRandomWords(new Random().nextInt(20, 30))  
                ).build();  
    }  
    public Image getImage(String filepath) {  
        return Image.newBuilder()  
                .setId(UUID.nameUUIDFromBytes(filepath.getBytes()).toString())  
                .setFilePath(filepath)  
                .setUploadedAt(fromMillis(currentTimeMillis()))  
                .build();  
    }  
    public Book.Genre randomGenre() {  
        String genre = genreList[rand.nextInt(genreList.length)];  
        if ("FICTION".equals(genre)) {  
            return Book.Genre.FICTION;  
        } else if ("MYSETRY".equals(genre)) {  
            return Book.Genre.MYSTERY;  
        } else if ("THRILLER".equals(genre)) {  
            return Book.Genre.THRILLER;  
        } else if ("HORROR".equals(genre)) {  
            return Book.Genre.HORROR;  
        } else if ("HISTORICAL".equals(genre)) {  
            return Book.Genre.HISTORICAL;  
        } else if ("ROMANCE".equals(genre)) {  
            return Book.Genre.ROMANCE;  
        } else if ("SCI_FICTION".equals(genre)) {  
            return Book.Genre.SCI_FICTION;  
        } else {  
            return Book.Genre.UNKNOWN;  
        }    }  
    public Book createRandomBook() {  
        String bookName = randomName();  
        return Book.newBuilder()  
                .setId(UUID.nameUUIDFromBytes(bookName.getBytes()).toString())  
                .setName(bookName)  
                .setAuthor(randomAuthor())  
                .setPrice(rand.nextInt(1, 300))  
                .setPublication(randomPublication())  
                .setPublishYear(rand.nextInt(1990, 2023))  
                .addSample(randomSample())  
                .addSample(randomSample())  
                .addImage(getImage("src/main/resources/images_source/img1.png"))  
                .addImage(getImage("src/main/resources/images_source/img2.png"))  
                .addImage(getImage("src/main/resources/images_source/img3.png"))  
                .addGenre(randomGenre())  
                .addGenre(randomGenre())  
                .addGenre(randomGenre())  
                .setPopularity(rand.nextInt(100))  
                .build();  
    }}
```

This would generate book with random fields and we could use it to create book in our service.