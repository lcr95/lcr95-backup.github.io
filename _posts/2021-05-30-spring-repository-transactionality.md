---
layout:     post
title:      "Spring Repository Transnationality"
subtitle:   "" 
date:       2021-05-30 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - Java
    - Spring	
    - JPA
---



Recently, I met the following exception when trying to create my custom delete method on repository.

> org.springframework.dao.InvalidDataAccessApiUsageException: No EntityManager with actual transaction available for current thread - cannot reliably process 'remove' call; nested exception is javax.persistence.TransactionRequiredException: No EntityManager with actual transaction available for current thread - cannot reliably process 'remove' call



**<u>Root Cause</u>**

By default Spring only mark the CRUD methods as transactional. Hence, the custom query method we defined does not inherit the transactional annotation.  



**<u>Solution</u>**

Just mark the custom method with `@Transactional ` .

```java
public interface ComponentDependencyRepository extends JpaRepository<UserProfile, String> {

    List<ComponentDependency> findByPhoneNo(String phoneNo);

    @Transactional
    void deleteByPhoneNo(String phoneNo);

}
```





<br>

**<u>Reference</u>**

[Stackoverflow](https://stackoverflow.com/q/39827054/2985850)

