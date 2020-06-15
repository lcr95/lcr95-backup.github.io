---
layout:     post
title:      "Using Custom Function in Ignite SQL"
subtitle:   ""
date:       2020-06-13 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags: 
    - Ignite
---

Apache Ignite: 2.8.1

In this example, we will create a custom function that can be used in SQL Ignite.

# Ignite Installation 
Refer [documentation](https://apacheignite.readme.io/v1.3/docs/getting-started) for more infomation.

*In this example, we will install Apache Ignite in a Window enviroment.*

# Create Custom Function 
We will create a Custom Function under a class called ``MyCustomFunction``.

In the class we will define a custom function called ``Is_Pass(int score)`` which wil return ``true`` when score more than 50; return ``false`` when less or equal to 50.

```java
package com.ignite.example.udf;

import org.apache.ignite.cache.query.annotations.QuerySqlFunction;

public class MyCustomFunction {

    @QuerySqlFunction
    public static boolean Is_Pass(int score) {
        if (score > 50) {
            return true;
        } else {
            return false;
        }
    }
}
```

Compile it into a jar. In our case, a jar named ``ignite-sql-custom-function-example-1.0.0.jar`` is generated.


# Register Custom Function

1. Move the generated jar that contained custom function into Ignite's libs folder ``<path>/apache-ignite-2.8.1-bin/libs``. 
<br>

2. Register the custom function in a template. 
Edit the configuration file ``<path>/apache-ignite-2.8.1-bin/config/default-config.xml`` as below:<br>

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

 <bean class="org.apache.ignite.configuration.IgniteConfiguration">
   <property name="cacheConfiguration">
       <list>
          <bean id="cache-template-bean" abstract="true" class="org.apache.ignite.configuration.CacheConfiguration">
            <property name="name" value="MyTemplate*"/>
            <property name="cacheMode" value="PARTITIONED" />
            <property name="backups" value="1" />
            <property name="sqlFunctionClasses" value="com.ignite.example.udf.MyCustomFunction"/>
          </bean>
       </list>
      </property>
 </bean>
</beans>
```

**Note: The template name have to end with '*', else the template will not be found in runtime.**

Source Code : [Github](https://github.com/lcr95/ignite-custom-sql-function-example)

# Run Ignite Server
Run the server with ``<path>/apache-ignite-2.8.1-bin/bin/ignite.bat``


# Execute Custom Function with SQL
In this example, we will use a tools called [DBeaver](https://dbeaver.io/). Refer [here](https://apacheignite-sql.readme.io/docs/sql-tooling), for more infomation on using it to interact with Ignite.
<br>
Use Case: We want to find out student who passed the exam.

1. Create table in Ignite with the template we register [earlier](#register-custom-function). 
```sql
CREATE TABLE Student (
  ID LONG PRIMARY KEY, NAME VARCHAR, SCORE INTEGER)
  WITH "template=MyTemplate"
```


2. Insert dummy data into table.
```sql
  INSERT INTO Student (ID, NAME,SCORE) VALUES (1, 'Johny', 80);
  INSERT INTO Student (ID, NAME,SCORE) VALUES (2, 'Mark' ,50);
  INSERT INTO Student (ID, NAME,SCORE) VALUES (3, 'Jason', 30);
```

3. Query student who is passed the exam:
```sql
SELECT * 
FROM Student
WHERE Is_Pass(score)
```
Result:
{% include image.html src="post-ignite-udf-result.png" data="group" title="Query Result" %}



