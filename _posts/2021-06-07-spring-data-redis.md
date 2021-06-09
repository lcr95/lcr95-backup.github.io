---
layout:     post
title:      "Spring Data Redis"
subtitle:   "" 
date:       2021-06-07 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Java
---



Redis is a famous in-memory data store that often used as database, cache, and message broker. Spring framework come out with [Spring Data Redis](https://spring.io/projects/spring-data-redis) that provide a very easy configuration and access to Redis. In this article, we will look into how to create a simple spring boot application that can:

- Read / Write data into Redis via Spring REST call
- Pub data via Spring REST call





# Redis Setup

Run Redis in docker.

`docker run -d -p 6379:6379 -v redis_data:/data --name rds redis`





# Maven Dependency

Make sure the following dependency is included in `pom.xml`.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.4.3</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.9.0</version>
</dependency>
```





# Spring Configuration

We need to configure the login credential for Spring Data Redis to access Redis.

e.g. sample configuration of `application.yaml`

```yaml
spring:
  redis:
    database: 0
    password: password123
    port: 6379
    host: localhost
    lettuce:
      pool:
        min-idle: 5
        max-idle: 10
        max-active: 8
        max-wait: 1ms
      shutdown-timeout: 100ms
```





# Data / Domain Structure 

Next, we will need to define the data / domain structure. Spring Data repository does not require us to have a deep knowledge about Redis. We only need to annotate the domain class properly and Spring will take care the rest of it. 

Assume, we have a JSON payload as below:

```json
{
    "id": "id123",
    "name": "Chen Riang",
    "type": "type1",
    "data": "some data"
}
```



We will create a Java POJO class to represent it.

```java
@Data
@Builder
@RedisHash("RedisData")
public class RedisData implements Serializable {
    @Id
    private String id;
    private String name;
    @Indexed
    private String type;
    private String data;
}
```

- The field with annotation `@Id` means that it is going to be the actual key that used to persist the value.
- The domain class have to annotate is as `@RedisHash` and this indicate that (in our case `RedisData` class object) are going to be the value that store in Redis key-value store.

- The annotation `@Indexed` will instruct Spring Data Redis to create a secondary index and to be use when we instruct a query.





# Spring Repository

Before implement write and read feature from Redis, we need to implement a repository which access and communicate with Redis. This is where the place we implement CRUD. 



Create repository as below:

```java
@Repository
public interface RedisDataRepository extends CrudRepository<RedisData, String> {

    
}
```

- It is ok to leave it empty if we only need some of the basic method e.g. `save` and `delete`. See [Javadoc](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) for more info. 



Next, we need to create a configuration class that used to create `RedisTemplate` bean object. 

```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {

    @Bean
    public RedisTemplate<String, RedisData> redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
        RedisTemplate<String, RedisData> template = new RedisTemplate<>();
        template.setConnectionFactory(lettuceConnectionFactory);
        return template;
    }
    
}
```

- We need to annotate `@EnableRedisRepositories` on the configuration or main class to activate Spring Data Redis.

- Although we do no need to use `RedisTemplate` at the moment but, we still need to declare it as it is used by CRUD repositories for integration with Redis.





# Write Data

To write the data into Redis, we can simply call the `save` method in Repository class like below:

```java
@Autowired
private RedisDataRepository redisDataRepository;

RedisData redisData; // assume this is initialized

redisDataRepository.save(redisData)

```



One of the objective of this article is to write data from a REST call. Thus, we will create an controller class and expose the API as below:

```java
@RestController
public class RedisController {

    @Autowired
    private RedisDataRepository redisDataRepository;
    
    @PostMapping("/redis/save")
    public ResponseEntity save(@RequestBody RedisData redisData) {
        redisDataRepository.save(redisData);
        return ResponseEntity.ok().build();
    }
}
```

- Spring will auto convert the JSON payload into RedisData object when we invoke the POST endpoint `/redis/save`.





# Read Data

To read the data, we will implement custom CRUD method to 

-  find by Id (`id`)
- find by index (`type`)



First, we need to add the respective method in the `RedisDataRepository` interface we created earlier.

```java
@Repository
public interface RedisDataRepository extends CrudRepository<RedisData, String> {

    Optional<RedisData> findById(String id);

    List<RedisData> findByType(String type);
}
```



After that, we will expose REST API for each function in our controller.

```java
@RestController
public class RedisController {
    
    @Autowired
    private RedisDataRepository redisDataRepository;
    
    ...
        
    @GetMapping("redis/read/type/{type}")
    public List<RedisData> readByType(@PathVariable String type) {
        return redisDataRepository.findByType(type);
    }

    @GetMapping("redis/read/id/{id}")
    public RedisData readById(@PathVariable String id) {
        return redisDataRepository.findById(id).get();
    }
    
}
```

- To find by id we can call the GET endpoint e.g. `redis/read/id/id123` if we want to find record of "id123".

- To find by type we can call the GET endpoint e.g. `redis/read/type/type1` if we want to find all record of type "type1".






# Pub Data 

To achieve another objective, Pub data via Spring REST call, we will use a method(`convertAndSend()`) in `RedisTemplate` to publish data.

```java
@RestController
public class RedisController {
    
    @Autowired
    private RedisTemplate<String, RedisData> redisTemplate;
    
    ...
        
    @PostMapping("/redis/pub")
    public ResponseEntity push(@RequestBody RedisData redisData) {
        String pubSubChannel = "redis-data";
        redisTemplate.convertAndSend(pubSubChannel, redisData);
        return ResponseEntity.ok().build();
    }
}
```

- To publish the data we need to invoke the endpoint `/redis/pub`.
- The data will publish to a channel called `redis-data`.





# Sub Data

To verify what we publish is working perfectly, we will create a Redis subscriber in the Springboot Application. 



First, we need to create our own custom Redis subscriber.

```java
@Service
public class RedisSubscriber implements MessageListener {

    @Autowired
    private RedisTemplate<String, RedisData> redisTemplate;

    @Override
    public void onMessage(Message message, byte[] bytes) {
        RedisSerializer<?> valueSerializer = redisTemplate.getValueSerializer();
        RedisData redisData = (RedisData) valueSerializer.deserialize(message.getBody());

        System.out.println("Data received from redis publisher");
        System.out.println("id   : " + redisData.getId());
        System.out.println("name : " + redisData.getName());
        System.out.println("type : " + redisData.getType());
        System.out.println("data : " + redisData.getData());
    }
}
```

- The subscriber will implement the Spring `MessageListener` interface.
- As we are publishing a pojo object, we will to get the `valueSerializer` object from `redisTemplate` to deserialize the message received.



Second, we need to define `MessageListenerAdapter` bean which inject our custom `RedisSubscriber` into it.

```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {
	
	...
	
	@Bean
    MessageListenerAdapter messageListener(RedisSubscriber redisSubscriber) {
        return new MessageListenerAdapter(redisSubscriber);
    }
}
```

- By specifying `RedisSubscriber` in the method argument, Spring will autowired `RedisSubscriber` object.



Next, we need to create a `ChannelTopic` bean to tell the listener which topic it should listened to.

```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {
	
	...
	
    @Bean
    ChannelTopic topic() {
        return new ChannelTopic("redis-data");
    }
}
```



Last, we need to create a `RedisMessageListenerContainer ` that provided by Spring Data Redis which ensure async behavior for Redis message listener.

```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {
	
	...

	@Bean
    RedisMessageListenerContainer redisContainer(LettuceConnectionFactory lettuceConnectionFactory,
                                                 MessageListenerAdapter adapter, ChannelTopic topic) {
        final RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(lettuceConnectionFactory);
        container.addMessageListener(adapter, topic);
        return container;
    }
}
```

- By specifying `LettuceConnectionFactory`, `MessageListenerAdapter`, and `ChannelTopic` Springboot will autowired correct object when the method is invoked.





# Conclusion

In this article, we exanimated how to use Spring Data Redis to do read/write and pub/sub.



The implementation of above example can be found [here](https://github.com/lcr95/redis-spring-data-example)(Github)



