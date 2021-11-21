---
layout: post
title: "Redis Server Side Scripting with Lua"
subtitle: ""
date: 2021-11-21
author: "ChenRiang"
header-style: text
tags:
    - Redis
    - Lua
    - Java	
    - Spring
---



Performance and light weight are always reasons why people want to use Redis. If the ordinary use of Redis does not fulfil your requirement, server side scripting with Lua might come into rescue. Redis let you create your own scripted extension with Lua script. This enables us to perform variety of operations in Redis, which can simplify our code and increase performance. In this blog post, we will look into how to execute Lua script in CLI and Java (Springboot)



To better illustrate, let's look into the following example:  We are going to consider Bank ABC use Redis to store the balance of 3 accounts. Each account balance information is stored in a Redis key with following format:

>  `<account type>:<customer id>`



<br/>

{% include image.html src="use-case.png" data="group" title="" %}



In this example we want to obtain the total balance of all 3 account own by `customer001`. The simplest solution will be communicating with Redis 3 times and sum all the return result. However, this will incur some latency wait time when calling Redis and we will further improve performance by executing all the logic in server side.



------

Setting up the scene. Run the following command line by line in Redis CLI

```bash
set savingAcc:customer001 500
set currentAcc:customer001 600
set fdAcc:customer001 5000
```

------

**Solution** 

The solution is pretty straightforward, move and execute the entire logic in server side. The Lua script expects users to pass in 1 key and 0-3 argument variable.

`key` = Customer Id

`ARGV` = Account Type

```lua
local customerId = KEYS[1];

local function getAccountBalance(accountId)
    local balance = redis.call('GET', accountId);
    if (balance == nil) then
        return 0;
    else
        return balance;
    end
end

local total = 0;
if (ARGV[1] ~= nil) then
total = total + getAccountBalance(ARGV[1] .. ':' .. customerId)
end

if (ARGV[2] ~= nil) then
total = total + getAccountBalance(ARGV[2] .. ':' .. customerId)
end

if (ARGV[3] ~= nil) then
total = total +  getAccountBalance(ARGV[3] .. ':' .. customerId)
end

return total;
```





# Redis CLI

To run Lua script using Redis CLI, we will use the [EVAL](https://redis.io/commands/eval) command. This command will use the Lua interpreter in Redis to evaluate the Lua script.  

Format:

`EVAL <script> <no of keys> <key[]> <arg[]> `

```
eval "local customerId = KEYS[1]; local function getAccountBalance(accountId) local balance = redis.call('GET', accountId); if (balance == nil) then return 0; else return balance; end end local total = 0; if (ARGV[1] ~= nil) then total = total + getAccountBalance(ARGV[1] .. ':' .. customerId) end if (ARGV[2] ~= nil) then total = total + getAccountBalance(ARGV[2] .. ':' .. customerId) end if (ARGV[3] ~= nil) then total = total +  getAccountBalance(ARGV[3] .. ':' .. customerId) end return total;" 1 customer001 savingAcc fdAcc currentAcc
```





# Springboot 

We will run this example as a Spring component and the Lua script (`"account_sum.lua"`)  above is saved in resources folder.

```java
@Component
public class RedisLuaExample {

    @Autowired
    RedisTemplate<String, String> redisTemplate;

    @SneakyThrows
    @PostConstruct
    public void init() {
        List<String> keys = Collections.singletonList("customer001");
        String[] args = new String[] {"savingAcc", "fdAcc", "currentAcc"};
        Long result = redisTemplate.execute(getScript(), keys, args);
        System.out.println("Total Amount : " + result.toString());
    }

    private static RedisScript<Long> getScript() {
        Resource scriptSource = new ClassPathResource("account_sum.lua");
        return RedisScript.of(scriptSource, Long.class);
    }
}

```

Check out the java source code [here](https://github.com/lcr95/java-example/blob/main/redis-spring-data-example/src/main/java/com/example/redis/lua/RedisLuaExample.java).



# Conclusion

Since, the main motivation of Lua script is to speed up the application's  performance. Should I move all my logic and execute them in Lua script? 



**No.** Lua server scripting might not 100% guarantee you a performance boost as it depends on your use case. Always run the benchmark performance test to prove that it gives you a better result before using them.



**Reference**

1. A Speed Guide To Redis Lua Scripting - [link](https://www.compose.com/articles/a-quick-guide-to-redis-lua-scripting/)

2. A quick guide to Redis Lua scripting - [link](https://www.freecodecamp.org/news/a-quick-guide-to-redis-lua-scripting/)

3. Redis Lua Script With Spring Boot - [link](https://www.vinsguru.com/redis-lua-script-with-spring-boot/)
