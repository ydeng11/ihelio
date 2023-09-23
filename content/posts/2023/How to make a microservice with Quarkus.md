---
title: "How to make a microservice with Quarkus"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Programming"
tags: 
    - "JAVA"
    - "Microservices"
    - "Quarkus"
---

## What is Quarkus

Quarkus is a full-stack, Kubernetes-native Java framework that was developed by Red Hat. It first appeared in early 2019, aimed at optimizing Java specifically for containers and enabling it to become an effective platform in serverless environments.

The motivation behind Quarkus was to breathe new life into the Java ecosystem for modern cloud-native applications. It seeks to overcome the traditional shortcomings of Java, like slow startup time and high memory consumption, which are particularly notable in containerized environments. Quarkus achieves this through ahead-of-time (AOT) compilation which drastically reduces the runtime memory overhead and speeds up the startup time, making Java a more competitive choice in the modern landscape of microservices and serverless architectures.

What makes Quarkus distinct, especially under the guidance of Red Hat, is its live coding feature which allows developers to see changes in real-time without the need for redeploys, enhancing the developer experience greatly. Additionally, Quarkus is designed to work seamlessly with popular Java standards, frameworks, and libraries, meaning that developers don't have to learn new APIs to start benefiting from it. It integrates effortlessly with Kubernetes and OpenShift, promoting a simplified and productive development process, especially in cloud-native environments.

Furthermore, it emphasizes unifying imperative and reactive programming models, bringing in a new paradigm that is more flexible and powerful, catering to a broader range of use-cases and accommodating a larger audience of developers who are familiar with different programming models.

In essence, with the development of Quarkus, Red Hat has propelled Java into the modern age, making it a strong contender in the cloud-native development space, fostering quicker, more efficient, and scalable application development.

## Why you should use Quarkus

we're talking about Quarkus, Spring Boot, and Micronaut. These guys are like the superheroes of the Java ecosystem, each with its own superpower!

**Spring Boot**, the seasoned veteran in the group, has been the go-to for building Java applications for quite a while now. It comes packed with a rich set of libraries and a massive community support, which makes it pretty robust and versatile. But, as it's quite mature, it can sometimes feel a bit too heavy, especially when you're trying to whip up microservices that are lean and mean.

Then enters **Micronaut**, which was like a breath of fresh air when it appeared on the scene. It came in swinging, offering quicker startup times and lower memory footprint compared to Spring Boot, thanks to its ahead-of-time (AOT) compilation. It kind of shook things up, showing that Java applications can indeed be lightweight and nimble.

Now, let's talk about the new kid on the block, **Quarkus**. Developed by Red Hat, it kind of took the Java world by storm around 2019. It's like it looked at what Spring Boot and Micronaut were doing and said, "Hold my coffee!" Quarkus further ramps up the efficiency game, offering blisteringly fast startup times and incredibly low memory usage. It's tailor-made for containerized environments like Kubernetes, making it a rockstar in the cloud-native development arena.

But here's where Quarkus really shines - its live coding feature is nothing short of a game-changer. Imagine being able to see the changes to your code in real-time, without those pesky redeploys. It's like having a conversation with your application, which seriously ramps up productivity.

So, why should you consider hitching your wagon to Quarkus? Well, it's kind of bringing sexy back to Java. It's lightweight, it's nimble, and it's just perfect for the modern, containerized world where resources are at a premium. Not to mention, it marries the best of both worlds – the imperative and reactive programming models – giving you the flexibility to code the way you like it.

In a nutshell, if you're looking to build microservices that are not just efficient but also fun to work with, you might want to give Quarkus a whirl. It's fresh, it's exciting, and it might just be the spark that reignites your passion for Java development.

So there you have it, a quick glimpse into the cool, modern world of Java frameworks. Each has its strengths, but Quarkus seems to be nudging ahead with its innovative approach to making Java fun and efficient again. Give it a shot, and you might just find it to be a breath of fresh air in your development adventures!
### Development

Quarkus provides a convenient way for local development, using `quarkus dev` command to start up the app in local with OpenAPI available to test the Rest endpoints. In addition, Quarkus allows us to use different `dev` profiles by specifying the `application.properties`, e.g. we could use `quarkus dev -Dquarkus.profile=dev`  to start the app using `application-dev.properties`. Thus we could customize the development environment with database connections or listening port.

Once the app starts up, the changes to the code would be lively deployed so we don't have to rebuild and redeploy. And it is very easy to add debugger to Quarkus ([# How to debug a Quarkus application with IntelliJ IDEA](https://www.youtube.com/watch?v=44Apvlumx0E)).

#### Dependency Injection

Quarkus using Jarkarta for DI and all it needs is adding the proper annotation like `@ApplicationScoped` or `@Singleton`. But the official site mainly uses `@ApplicationScoped`. This is a great [article](https://marcelkliemannel.com/articles/2021/overview-of-bean-scopes-in-quarkus/) explaining the different scopes in Quarkus.

#### Exception Handler

Quarkus also supports exception handler shown as below:
```java
@Provider
public class CustomExceptionHandler implements ExceptionMapper<CustomException> {
  private final static Logger logger = LoggerFactory.getLogger(CustomExceptionHandler.class);

  @Override
  public Response toResponse(CustomException e) {
    logger.error("cannot complete the request.", e);
    if (e.getCause() instanceof RecordAlreadyExistingException
        || e.getCause() instanceof NotFoundException
        || e.getCause() instanceof IllegalArgumentException) {
      return Response.status(Response.Status.BAD_REQUEST)
          .entity(new CustomException.ErrorResponseBody(e.getCause().getMessage()))
          .build();
    } else {
      return Response.status(Response.Status.INTERNAL_SERVER_ERROR)
          .entity(new CustomException.ErrorResponseBody("Something unexpected happened. Try again"))
          .build();
    }
  }
}
```

We could replace `CustomException` with any exception type we want to further process and map to a response.
#### Health Check

Health check is also easy. We just need to add the `Liveness` annotation and implements the interface `HealthCheck`. Quarkus will provide the health check endpoint at `{http://localhost:8080}/q/health/live`. 

```java
@Liveness
@ApplicationScoped
public class SimpleHealthCheck implements HealthCheck {
  @Override
  public HealthCheckResponse call() {
    return HealthCheckResponse.up("I am healthy");
  }
}
```

#### Database Connection

You could use Hibernate via Panache if you want to use ORM ([tutorial](https://quarkus.io/guides/hibernate-orm-panache)). But I personally like using Jooq to interact with database. And it is also very easy ([example POM](https://github.com/ydeng11/Minance/blob/33f9dac0cacacc46ec0bdf406e6dc51c1e0c4142/pom.xml#L217-L287)):
1. Install the required dependency
	1. flyway
	2. Jooq
2. Generate the Jooq records
3. `DSLContext` could be directly inject
### Test

The best part I love with Quarkus is its `devservice` which is a built-in test container allowing us to test the transactions. We could set up the database connection in the `application.properties` in test folder or a separate test profile class which could be specified for a particular test class. In below, I set up the MySQL db for `devservice` and `flyway` to create the schemas for transaction tests.

```bash
quarkus.datasource.db-kind=mysql  
quarkus.datasource.devservices.db-name=minance  
quarkus.datasource.devservices.username=quarkus  
quarkus.datasource.devservices.password=quarkus  
quarkus.datasource.devservices.port=30301  
quarkus.flyway.enabled=true  
quarkus.flyway.clean-at-start=true  
quarkus.flyway.migrate-at-start=true  
quarkus.flyway.username=quarkus  
quarkus.flyway.password=quarkus  
quarkus.flyway.jdbc-url=jdbc:mysql://127.0.0.1:30301/minance?createDatabaseIfNotExist=true  
quarkus.flyway.default-schema=minance
```

### Build & Deploy

Since Quarkus is meant to be a cloud native framework, it comes with several `Dockerfile` for building a docker image. For example, we could firstly run `quarkus build` to build an exectuatble jar and build a docker image using: `docker build --platform linux/amd64 -f src/main/docker/Dockerfile.jvm -t repo/image:0.0.1 .` for amd64 platform. 

Then we could deploy it by running `docker run -p 8080:8080 repo/image:0.0.1` to start the container.