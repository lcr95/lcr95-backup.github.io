---
layout: post
title: "Springboot autowire in non-bean object"
subtitle: ""
date: 2021-07-02
author: "ChenRiang"
header-style: text
tags:
    - Java
    - Spring
---



In Spring there is a feature **autowiring**, that will auto resolve and inject object dependency. With the help of autowriring, we does not need to explicitly inject the dependency and reduce the amount of code we need to write. However, simplicity doesn't come free, once a class is annotated to be Spring bean, Spring will take control of that particular class's object lifecycle (from creation to destroy) and we will lose some control towards it. 



So, this problem bug me for a while as I want to have full control towards my object but at the same time enjoy the autowring feature. In Spring, there is a class [AutowireCapableBeanFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/AutowireCapableBeanFactory.html) that could help us to achieve this.  



This is the class we want to have full control toward it.

```java
public class MyObject {
    
    @Autowired
    private ServiceA serviceA;
    
    @Autowired
    private RepositoryB repositoryB;
    
    ...
}
```



We can simply autowire the dependency as below:

```java
@Component
public class MyObject {
    @Autowired
    private AutowireCapableBeanFactory autowireCapableBeanFactory;
    
    
    public MyObject intiMyObject() {
        MyObject myObject = new MyObject();
        autowireCapableBeanFactory.autowireBean(myObject);
        return myObject;
    }
}
```



