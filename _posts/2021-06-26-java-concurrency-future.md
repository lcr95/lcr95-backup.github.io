---
layout: post
title: "Java Concurrency - Future"
subtitle: ""
date: 2021-06-26
author: "ChenRiang"
header-style: text
tags:
    - Java
    - Concurrency
---



When we talk about Concurrency topic in Java, for sure you will come across to this term, Future. In this article, we are going to learn about what is Future, and how can we use it.





# java.util.concurrent.Future

[Future](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html) as the name itself implies, it is a container that store a “future” result which does not exist yet. It is a class that represents a further result of an asynchronous computation. In a long running or computation heavy task, the use of asynchronous processing and Future can be super useful. We can encapsulate the task in Future and run some other process while waiting for the task to be completed.



# Implement Future 

In this example we are going to stimulate the long running task by creating a task that sleeps for 1 second and return some string.

```java
public static Future<String> longRunningTask() {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        return executorService.submit(() -> {
            Thread.sleep(1000);
            return "Value from future";
        });
    }
```





# Consume Future

There is two interesting API in Future class: isDone() and get()

### isDone()

Again, the name itself implies, `isDone()` will tell us if the task has completed. It will return `true` if completed, otherwise `false`.

```java
while(!future.isDone()) {
	System.out.println("Waiting future task to be completed");
	Thread.sleep(500);
}
```



### get()

This method will return result from the completed task and will block the thread execution if the task is not completed.

```java
String result = future.get();

System.out.println(result);
```



Besides, the `get()` also have an overloaded method that allow us to specify timeout . If the task does not return before the specified timeout, a `TimeoutException` will be thrown. 

```java
String result = future.get(200, TimeUnit.MILLISECONDS);
System.out.println(result);
```

<u>Example:</u>

```
Exception in thread "main" java.util.concurrent.TimeoutException
	at java.util.concurrent.FutureTask.get(FutureTask.java:205)
	at com.example.jmeter.test.SimpleFutureExample.main(SimpleFutureExample.java:23)
```



# Cancel Future

There is some case, we started the task and for some reason, we do not need the result anymore. We can simply cancel the task by calling `cance(boolean)` method. This method does not guarantee the task to be successfully cancelled the task can be already completed / cancelled / cannot be cancelled. The `cance(boolean)` will only attempt to cancel the task execution, and return `true` if successfully cancel, otherwise `false`. The method takes in a boolean argument which serve as a flag to let the program know should the task be interrupted or it is allowed to be completed. After the task is successfully cancelled, the subsequent call from `get()` will throw a `CancellationException`. Thus, calling the method `isCancelled()` can prevent us getting the exception.



```java
future.cancel(true);

String result = future.get();
System.out.println(result);
```

 <u>Example:</u>

```
Exception in thread "main" java.util.concurrent.CancellationException
	at java.util.concurrent.FutureTask.report(FutureTask.java:121)
	at java.util.concurrent.FutureTask.get(FutureTask.java:192)
	at com.example.jmeter.test.MainClass.main(MainClass.java:25)
```



# Conclusion

In this article, we discussed the usage of Future class. 



Check out the source code [here](https://github.com/lcr95/java-example/tree/main/concurrency-future-example).
