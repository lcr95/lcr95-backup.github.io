---
layout:     post
title:      "Groovy ExecutorService submit Callable"
subtitle:   "the little annoying thing" 
date:       2021-06-20 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Groovy
    - Java
    - Concurrency

---



I have been using Spock instead of JUnit for a few months now. Today, I’m trying something around thread safety and I found something about Groovy that annoyed me. In my unit test, I need to submit a `Callable` task to `ExecutorService`. 



# The Problem

In Java, normally we will normally do this:

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Callable<Integer> c = () -> 1 + 2;
Future<Integer> future = executor.submit(c);
System.out.println("Result: " + future.get());
```

<u>Output:</u>

```
Result: 3
```



---



In Groovy(Spock), 

```groovy
ExecutorService executor = Executors.newSingleThreadExecutor()
Callable<Integer> c = { -> return 1 + 2 }
Future<Integer> future = executor.submit(c)
System.out.println("Result: " + future.get())
```

<u>Output:</u>

```
Result: null
```



Why??!?



**<u>Root Cause</u>**

In Groovy, the closure is always default as `Runnable` instead of `Callable`. (See more in ticket [GROOVY-3295](https://issues.apache.org/jira/browse/GROOVY-3295)) Thus, when we execute `submit` method Groovy will auto convert it to `submit(Runnable)` instead of `submit(Callable)`. 



This happen even I assigned it to a variable `c` with type `Callable<Integer>`.



# The Solution



Use `as Callable`. 

```groovy
ExecutorService executor = Executors.newSingleThreadExecutor()
Callable<Integer> c = { -> return 1 + 2 }
Future<Integer> future = executor.submit(c as Callable<Integer>)
System.out.println("Result: " + future.get())
```

<u>Output:</u>

```
Result: 3
```



