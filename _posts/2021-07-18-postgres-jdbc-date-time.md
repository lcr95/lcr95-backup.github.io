---
layout: post
title: "Postgres JDBC - Date Time"
subtitle: ""
date: 2021-07-18
author: "ChenRiang"
header-style: text
tags:
    - Java
    - Postgres
---



Storing date-time data is not straightforward as we think when the software will be running in different region. Often when dealing date-time data with JDBC, we will use [`Date`](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Date.html), [`Time`](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Time.html), and [`Timestamp`](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Timestamp.html) from [`java.sql`](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/package-summary.html) which extends from the [` java.util.Date`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Date.html). However, in Java 8 a whole new set of date-time API, [`java.time`](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html) is introduced to replace the legacy(Java 7 and before) date-time API.

In this article, we will be looking into the how Postgres JDBC driver support for Java's legacy and modern date time API.



**<u>TL;DR</u>**

| **PostgreSQL**                 | **Legacy Java**      | **Modern Java**            |
| ------------------------------ | -------------------- | -------------------------- |
| DATE                           | `java.sql.Date`      | `java.time.LocalDate`      |
| TIME [ WITHOUT TIMEZONE ]      | `java.sql.Time`      | `java.time.LocalTime`      |
| TIMESTAMP [ WITHOUT TIMEZONE ] | `java.sql.Timestamp` | `java.time.LocalDateTime`  |
| TIMESTAMP WITH TIMEZONE        | `java.sql.Timestamp` | `java.time.OffsetDateTime` |

- `Timestamp` does not have UTC awareness, but `OffsetDateTime` has.

# DB Setup

Create a SQL table that consist of date, timestamp and time column for later example use. 

```sql
CREATE TABLE public.date_example (
	id varchar NOT NULL,
	my_date date NULL,
	my_timestamp timestamp NULL,
	my_time time NULL
);
```



# Implementation 

### Legacy Java (Before Java 7)

Insert data into database:

```java
 private static void insertWithJavaSqlAPI() {
        try (Connection connection = DriverManager.getConnection(POSTGRE_URL, "postgres", "postgres")) {
            String insertSql = "INSERT INTO public.date_example (id, my_date, my_timestamp, my_time) VALUES(?,?,?,?)";
            PreparedStatement insertStatement = connection.prepareStatement(insertSql);

            insertStatement.setObject(1, "id2");
            insertStatement.setObject(2, new Date(System.currentTimeMillis()));
            insertStatement.setObject(3, new Timestamp(System.currentTimeMillis()));
            insertStatement.setObject(4, new Time(System.currentTimeMillis()));
            insertStatement.executeUpdate();

        } catch (Exception e) {
            log.error("error", e);
        }
    }
```





Select inserted data from database:

```java
private static void getResultSetWithJavaSqlAPI() {
        try (Connection connection = DriverManager.getConnection(POSTGRE_URL, "postgres", "postgres")) {
            String selectSql = "SELECT * FROM public.date_example WHERE id = ?";
            PreparedStatement selectStatement = connection.prepareStatement(selectSql);

            selectStatement.setObject(1, "id2");
            ResultSet resultSet = selectStatement.executeQuery();
            while (resultSet.next()) {
                Date my_date = resultSet.getDate("my_date");
                System.out.println(my_date.toString());

                Timestamp my_timestamp = resultSet.getTimestamp("my_timestamp");
                System.out.println(my_timestamp.toString());

                Time my_time = resultSet.getTime("my_time");
                System.out.println(my_time.toString());
            }
        } catch (Exception e) {
            log.error("error", e);
        }
    }
```





### Modern Java (After Java 8)

Insert Data into table:

```java
 private static void insertWithJavaTimeAPI() {
        try (Connection connection = DriverManager.getConnection(POSTGRE_URL, "postgres", "postgres")) {
            String insertSql = "INSERT INTO public.date_example (id, my_date, my_timestamp, my_time) VALUES(?,?,?,?)";
            PreparedStatement insertStatement = connection.prepareStatement(insertSql);

            insertStatement.setObject(1, "id1");
            insertStatement.setObject(2, LocalDate.now());
            insertStatement.setObject(3, OffsetDateTime.now());
            insertStatement.setObject(4, LocalTime.now());
            insertStatement.executeUpdate();

        } catch (Exception e) {
            log.error("error", e);
        }
    }
```





# Why Should I Care?

This is the question you might ask. Since the old implementation is working fine, why should I care?

Here's why:

- `Timestamp` does not have UTC awareness, but `OffsetDateTime` does.

  Below is the result obtain from select result statement using both API:

  ```text
  Java Time API
  my_date = 2021-07-18
  my_timestamp =2021-07-18T14:41:46.544Z
  my_time =14:41:46.544
  
  Java Sql API
  my_date = 2021-07-18
  my_timestamp =2021-07-18 15:00:06.762
  my_time =15:00:06
  ```

    

  Notice that the result print out by `OffsetDateTime` by default already is UTC format. This isn't a big deal to you if your application does not concern about time zone issue. If your application has to deploy in different region, you should always store your timestamp in UTC format to avoid confusion about time zones and daylight saving time issue. **Thus, by using `OffsetDateTime`, we do not need to include the UTC date conversion logic in your code.**

<br>



- `java.time` API has more flexibility and utility method that process date time related action. 

  e.g.

  we wanted minus 2 days from current timestamp.

  ```java
  OffsetDateTime.now().minus(2, ChronoUnit.DAYS);
  ```





# Conclusion

In this article, we look into the implementation of legacy and modern java date time API in Postgres JDBC. Although the old legacy Java API still works perfectly, migrating to the new Java date time API is definitely the way to go     
