---
title: Global Exception Handling in Spring Boot
date: 2024-09-16 18:01:40
tags: 
  - Spring Boot
  - Exception Handling
categories:
  - [Spring Boot, Exception Handling]
cover: https://pics.findfuns.org/exception-handling.png   
---
# Introduction

When building projects with Spring Boot, handling complex business requirements often involves dealing with various exceptions. For example, during file I/O operations, you may encounter `IOException` or `FileNotFoundException`. When writing SQL statements or using JDBC, you may face `SQLException`. When working with reflection-related code, `ClassCastException` may occur.

In addition, there are many common exceptions such as `NullPointerException`, `ArrayIndexOutOfBoundsException`, `ConcurrentModificationException` (which occurs when modifying a collection while iterating over it), and arithmetic exceptions like division by zero (`ArithmeticException`), and so on.

When handling these exceptions, there are generally two approaches. The most straightforward and convenient way is to use the `throws` keyword to propagate the exception and let the upper-level method handle it. Another way is to catch exceptions using try-catch blocks. However, both approaches have obvious drawbacks. As the project grows larger and the number of exception handling points increases, handling exceptions one by one becomes inefficient and difficult to manage in a unified way.

So, is there a way to manage exceptions globally?

### `RestControllerAdvice` and `ExceptionHandler`

Classes annotated with `@RestControllerAdvice` can be used to handle global exceptions. By annotating methods with `@ExceptionHandler`, you can specify which type of exception the method handles.

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(RuntimeException.class)
    public Response exceptionHandler(RuntimeException e) {
        log.error("Internal Server Error " + e);
        return Response.error(e.getMessage(), ExceptionEnum.INTERNAL_SERVER_ERROR.getCode());
    }
}
```

`@ExceptionHandler` accepts parameters of type `Class[]`, representing the types of exceptions it can handle.

Define a basic exception interface and an enumeration class:

```java
public interface BaseException {

    String getCode();
    String getMsg();
}
```

```java
public enum ExceptionEnum implements BaseException {
    SUCCESS("200", "Success"),
    BAD_REQUEST("400", "Bad Request"),
    NOT_FOUND("404", "Not Found"),
    METHOD_NOT_ALLOWED("405", "Method Not Allowed"),
    INTERNAL_SERVER_ERROR("500", "Internal Server Error");

    private final String code;
    private final String msg;

    ExceptionEnum(String code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    @Override
    public String getCode() {
        return this.code;
    }

    @Override
    public String getMsg() {
        return this.msg;
    }
}
```

Define a `Response` class to unify the response format:

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Response {

    private String msg;

    private String code;

    public static Response success(ExceptionEnum exceptionEnum) {
        Response response = new Response(exceptionEnum.getMsg(), exceptionEnum.getCode());
        return response;
    }

    public static Response error(ExceptionEnum exceptionEnum) {
        Response response = new Response(exceptionEnum.getMsg(), exceptionEnum.getCode());
        return response;
    }

    public static Response error(String msg, String code) {
        Response response = new Response(msg, code);
        return response;
    }

    public static Response error(String msg) {
        Response response = new Response(msg, "-1");
        return response;
    }
}
```

Let’s test it by intentionally creating an arithmetic exception in the controller:

```java
@PutMapping("/add")
public String addPerson(@RequestBody Person person) {
    int i = 1 / 0;
    return myService.addPerson(person);
}
```

Send a request in Postman:

<img src="https://pics.findfuns.org/globalExceptionHandler.png" style="zoom:33%;" />

You can see that the returned `msg` is `"/ by zero"`, which corresponds exactly to the arithmetic exception, and the `code` is the predefined `500`.

In this way, by setting up global exception handling, we can centrally manage all exceptions in the project without handling them one by one. This allows us to focus entirely on business logic, which is much more convenient.