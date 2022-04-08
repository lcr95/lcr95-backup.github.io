---
layout: post
title: "Redis -  ERR EXEC without MULTI exception"
subtitle: ""
date: 2021-11-24
author: "ChenRiang"
header-style: text
tags:
    - Redis
    - Java
    - Spring

---



Today, I hit an exception when I tried to code my Redis logic to be transactional.

> io.lettuce.core.RedisCommandExecutionException: **ERR EXEC without MULTI**

Below is my code implementation.

```java
redisTemplate.opsForHash().put("customer001", "salary", "5000");
redisTemplate.watch("customer001");
redisTemplate.multi();
redisTemplate.opsForHash().put("customer001", "age", "20");
redisTemplate.exec();
```

From the code you will notice that I actually used `multi()` but somehow the framework not able to detect. I read from few blog stated transactional support is disabled by default in Spring Redis (*see the source code below*) and  `setEnableTransactionSupport(true)` will solve the issue.

*[RedisTemplate.java](https://github.com/spring-projects/spring-data-redis/blob/main/src/main/java/org/springframework/data/redis/core/RedisTemplate.java#L92)*

```java
public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {

    private boolean enableTransactionSupport = false;

    ....
```

***

However, I still get the same exception after I set  `setEnableTransactionSupport(true)`

```java
redisTemplate.opsForHash().put("customer001", "salary", "5000");
redisTemplate.setEnableTransactionSupport(true);
redisTemplate.watch("customer001");
redisTemplate.multi();
redisTemplate.opsForHash().put("customer001", "age", "20");
redisTemplate.exec();
```

***

**Solution**

I managed to fix the issue by using `SessionCallback` provided by Spring Data Redis. It is an interface for use when multiple operations need to be performed with the same connection.

```java
redisTemplate.execute(new SessionCallback<List<Object>>() {
    @Override
    public <K, V> List<Object> execute(RedisOperations<K, V> operations) throws DataAccessException {
        redisTemplate.opsForHash().put("customer001", "salary", "5000");
        redisTemplate.watch("customer001");
        redisTemplate.multi();
        redisTemplate.opsForHash().put("customer001", "age", "20");
        return redisTemplate.exec();
    }
});
```

***Note: This solution only work for Redis standalone mode as cluster mode does not support command like `watch`. Execute the logic using Lua script could be the solution for cluster mode.*** 

***

**Reference**

1.  Spring Redis Documentation - [link](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#tx.spring)

2.  关于RedisTemplate的ERR EXEC without MULTI错误 - [link](https://blog.csdn.net/Yihchu/article/details/108481675)
