---
layout: post
title: "Delete keys matching a specific pattern in Redis"
subtitle: ""
date: 2022-03-13
author: "ChenRiang"
header-style: text
tags:
    - Redis
    - Java
---

Deleting key(s) in Redis is pretty straightforward, you only need to issue a `DEL` command. However, `DEL` command can only remove the key(s) that exact match stated in the command, it does not support deleting key(s) that matched a pattern. In this blog post, we are going to look into how to delete multiple keys matching a specific pattern in Redis and execute them using Java Springboot.



# Delete via Lua script

The idea to achieve the objective is very simple, **search and delete**. We will create a Lua script to search for key that matched the given pattern and delete the response return. 

```lua
local cursor="0";

repeat
 local scanResult = redis.call("SCAN", cursor, "MATCH", "<Search key pattern>");
    local keys = scanResult[2];
    for i = 1, #keys do
        local key = keys[i];
        redis.call("DEL", key);
    end;
    cursor = scanResult[1];
until cursor == "0";
```

We used `SCAN` command instead of `KEYS` which return the similar result. The difference of these two commands is that, `KEYS` will scan all the keys in Redis with the provided matching pattern in a single go; `SCAN` will return a small number of search results and a cursor which indicate the current search position. With `SCAN` we can do scan incrementally without blocking the server traffic. We will loop the result with `DEL` command until there is no more further result (`cursor = "0"`)

# Execute in single node 

We will run the Lua script with Java Spring Data. See [Spring Data Redis](({% post_url 2021-06-07-spring-data-redis %})) to understand the setup. 

<u>cleanup.lua</u>

```lua
local pattern= ARGV[1];

local cursor="0";

repeat
    local scanResult = redis.call("SCAN", cursor, "MATCH", pattern);
    local keys = scanResult[2];
    for i = 1, #keys do
        local key = keys[i];
        redis.call("DEL", key);
    end;
    cursor = scanResult[1];
until cursor == "0";

return "";
```

<u>Java</u>

```java
 public class RedisService {
     @Autowired
     private RedisTemplate<String, String> redisTemplate;

     public void cleanup(String pattern) {
        Resource scriptSource = new ClassPathResource("cleanup.lua");
        RedisScript<String> redisScript = RedisScript.of(scriptSource, String.class);
        redisTemplate.execute(redisScript, Collections.emptyList(), pattern);
    }
 }
```



# Execute in cluster

You might notice some of the keys is not clear up when running above script on a Redis cluster. This is because `redisTemplate` will only send the script to randomly to only 1 node if we did not provide any key. A Redis cluster is divided up among 16,384 slots and these hash slots are a logical division of the keys.

e.g. In a 4 node cluster, the layout of slot will be :

| Slot          | Node |
| ------------- | ---- |
| 0 - 4095      | 0    |
| 4096 - 8191   | 1    |
| 8192 - 12287  | 2    |
| 12288 - 16384 | 3    |

To force the script to be run in every node, we need to get all the keys that belong to each corresponding node. 



<u>cleanup.lua</u>

```lua
local pattern= ARGV[1];

local cursor="0";

repeat
    local scanResult = redis.call("SCAN", cursor, "MATCH", pattern);
    local keys = scanResult[2];
    for i = 1, #keys do
        local key = keys[i];
        redis.call("DEL", key);
    end;
    cursor = scanResult[1];
until cursor == "0";

return "";
```



<u>Java</u>

```java
 public class RedisService {
     @Autowired
     private RedisTemplate<String, String> redisTemplate;

     public void cleanup(String pattern) {
        Resource scriptSource = new ClassPathResource("cleanup.lua");
        RedisScript<String> redisScript = RedisScript.of(scriptSource, String.class);
         
        RedisConnectionFactory connectionFactory = redisTemplate.getConnectionFactory();
        for (RedisClusterNode clusterNode : connectionFactory.getClusterConnection().clusterGetNodes()) {
            String key = redisTemplate.opsForCluster().randomKey(clusterNode);
            redisTemplate.execute(redisScript, Collections.singletonList(key), pattern);
        }
    }
 }
```



# Conclusion

In this post, we have looked into how to delete Redis key according to a given pattern and how to execute them using Java.





**<u>Reference</u>**

Ways to delete multiple keys from Redis cache - [link](https://medium.com/geekculture/how-to-delete-multiple-keys-from-redis-cache-252275a95579)

