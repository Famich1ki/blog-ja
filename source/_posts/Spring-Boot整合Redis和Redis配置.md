---
title: Spring Boot Integration with Redis and Redis Configuration
date: 2024-09-02 17:01:21
tags: 
  - Spring Boot 
  - Redis
  - JSON
categories:
  - [Spring Boot, Redis]
cover: https://pics.findfuns.org/springboot-redis.png
---
## Introduction

When talking about NoSQL databases, caching systems, message queues, token-based authentication, and similar use cases, Redis is almost always part of the discussion.

As an in-memory data structure store written in C, Redis is widely known for its high performance and simplicity. Thanks to its single-threaded design, it avoids lock contention issues common in multi-threaded environments, enabling it to handle hundreds of thousands of read and write operations per second.

In addition, Redis provides distributed solutions such as master–replica mode, Sentinel mode, and Cluster mode. These features improve availability and scalability while maintaining high performance, making Redis an indispensable component in modern application development.

---

## Integrating Redis with Spring Boot

With the help of various starters in Spring Boot, configuring Redis becomes quite straightforward.

### Step 1: Add Dependencies in `pom.xml`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

- `spring-boot-starter-data-redis` is the Spring Boot dependency for Redis. It includes the default Redis client **Lettuce**.
- Redis clients can be either **Lettuce** or **Jedis**. Lettuce is the default, supports asynchronous operations, and performs better in concurrent scenarios. It is generally recommended.
- `jackson-databind` is used for configuring Redis serialization later.
- `commons-pool2` is used when configuring Redis clusters.

---

## Redis Configuration

```java
@Configuration
public class RedisConfig {

    @Bean(name = "myRedisTemplate")
    public RedisTemplate<String, Object> getRedisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
        // Create a RedisTemplate instance to encapsulate Redis operations
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();

        // Configure connection factory
        redisTemplate.setConnectionFactory(lettuceConnectionFactory);

        // Custom serializers
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = getSerializer();

        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.setValueSerializer(genericJackson2JsonRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);
        redisTemplate.setHashValueSerializer(genericJackson2JsonRedisSerializer);

        return redisTemplate;
    }

    public GenericJackson2JsonRedisSerializer getSerializer() {
        ObjectMapper objectMapper = new ObjectMapper();

        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);

        objectMapper.activateDefaultTyping(
                LaissezFaireSubTypeValidator.instance,
                ObjectMapper.DefaultTyping.NON_FINAL
        );

        return new GenericJackson2JsonRedisSerializer(objectMapper);
    }
}
```

---

## Serialization Configuration

Before customizing Redis serialization, we should first understand why customization is necessary. Can Redis work without explicitly configuring a serializer?

Yes, it can.

By default, `RedisTemplate` and `RedisCache` use `JdkSerializationRedisSerializer`, which relies on Java’s built-in serialization mechanism (classes implementing `Serializable`).

However, there is a downside.

When data is stored in Redis using JDK serialization, the stored value is in binary format and not human-readable. This makes debugging and inspection more difficult.

For example, if we store a key-value pair `("k1", "v1")` using the default serializer and inspect it via `redis-cli`, we will see a binary-encoded value with hexadecimal prefixes added by the JDK serialization mechanism.

To retrieve the original value, we must use:

```java
redisTemplate.opsForValue().get("k1");
```

---

## Custom Serialization Options

Two commonly used serializers:

- `GenericJackson2JsonRedisSerializer`
- `Jackson2JsonRedisSerializer`

Both can correctly serialize:

- String
- Object
- Collections
- JSONObject
- JSONArray

### Key Difference

When using `GenericJackson2JsonRedisSerializer`, an additional `@class` property is added to each serialized object. The value of this property is the fully qualified class name.

`Jackson2JsonRedisSerializer` does not include this `@class` metadata. As a result, forced type casting may cause errors during deserialization.

For this reason, `GenericJackson2JsonRedisSerializer` is generally recommended.

---

## ObjectMapper Configuration

An `ObjectMapper` must be configured for `GenericJackson2JsonRedisSerializer`; otherwise, serialization of `JSONObject` may fail.

```java
objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
```

The `setVisibility` method configures the access level for fields and methods during serialization.

```java
objectMapper.activateDefaultTyping(
    LaissezFaireSubTypeValidator.instance,
    ObjectMapper.DefaultTyping.NON_FINAL
);
```

`activateDefaultTyping` enables proper serialization and deserialization when handling polymorphic types.

- `LaissezFaireSubTypeValidator` is a permissive type validator.
- `instance` is a static member returning a singleton instance.
- `DefaultTyping.NON_FINAL` specifies that type information should be included for all non-final classes.

---

## Encapsulating a RedisUtils Class

```java
@Component
public class RedisUtils {

    private final RedisTemplate<String, Object> redisTemplate;

    @Autowired
    public RedisUtils(@Qualifier("myRedisTemplate") RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public void set(String key, Object value) {
        redisTemplate.opsForValue().set(key, value);
    }

    public Object get(String key) {
        return redisTemplate.opsForValue().get(key);
    }

    public Set<String> getAllKeys() {
        return redisTemplate.keys("*");
    }
}
```

This utility class provides simple `set`, `get`, and `getAllKeys` methods.

---

## Test Class

```java
@SpringBootTest
@Slf4j
class RedisUsageApplicationTests {

    @Autowired
    RedisUtils redisUtils;

    @Test
    void contextLoads() throws JSONException {
        // Java POJO
        String JavaPojo = "JavaPojo";
        Person person01 = new Person("zzb", "English");
        redisUtils.set(JavaPojo, person01);

        // JSONObject
        String JSONObjectKey = "JSONObjectKey";
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("name", "zzb");
        jsonObject.put("age", 23);
        redisUtils.set(JSONObjectKey, jsonObject);

        // String
        String JavaString = "JavaString";
        redisUtils.set(JavaString, "v1");

        // List
        String JavaList = "JavaList";
        List<Person> list = new ArrayList<>();
        list.add(person01);
        redisUtils.set(JavaList, list);

        // JSONArray
        String JSONArrayKey = "JSONArray";
        JSONArray jsonArray = new JSONArray();
        jsonArray.put(person01);
        redisUtils.set(JSONArrayKey, jsonArray);

        log.info("JavaPojo -> {}", redisUtils.get(JavaPojo));
        log.info("JavaString -> {}", redisUtils.get(JavaString));
        log.info("JSONObject -> {}", redisUtils.get(JSONObjectKey));
        log.info("ArrayList -> {}", redisUtils.get(JavaList));
        log.info("JSONArray -> {}", redisUtils.get(JSONArrayKey));

        log.info("All keys -> {}", redisUtils.getAllKeys());
    }
}
```

