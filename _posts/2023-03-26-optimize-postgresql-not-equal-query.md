---
layout: post
title: "Optimize SQL Queires with Not-Equal Operator"
subtitle: ""
date: 2023-02-26
author: "ChenRiang"
header-style: text
tags:
    - Postgres
    - Performance
---
When working with a large volume of data in a database, it is important to optimize queries for performance. Recently, I faced a challenge in optimizing a "not equal" (<> or !=) search, which was very slow and caused the database to become unresponsive when the query was issued. This operator does not utilize an index, as it requires a full database scan in order to find matching records. In this blog post, we will explore ways to optimize this type of query.


My Postgres database table which contain 1 billion records with size 155Gb. 

{% include image.html src="table-size.png" data="group" title="" %}


This single SQL query killed the database.

```sql
SELECT COUNT(*) FROM alert
WHERE decision_alert_id <> 'CLN123'
```


# Using range operation

We can re-write the query to become:
```sql 
SELECT COUNT(*) FROM alert
WHERE decision_alert_id > 'CLN123' 
OR decision_alert_id < 'CLN123' 
```

This is a trick suggested by [araqnid](https://stackoverflow.com/a/2866129) on Stack Overflow. Postgres will consider utilizing the index if the selective range determined is very high. With this trick, my query completed in just 19ms.

{% include image.html src="result.png" data="group" title="" %}
{% include image.html src="result2.png" data="group" title="" %}


# Using partial index 
A partial index is an index created on a subset of rows in a table, based on a specific condition. This can significantly reduce the amount of data that needs to be scanned when performing a "not equal" search.

Below is the SQL to create partial index:
```sql
CREATE INDEX decisionAlertIndex 
ON alert (decision_alert_id) 
WHERE decision_alert_id <> 'CLN123';
```

Creating a partial index can achieve similar results as the previous trick. However, in my case, the input value used by the "not equal" operator is dynamic, making it infeasible to create a partial index.



# Conclusion 
In most cases, the "not equal" operator will not provide good performance when querying a large table, and we should try to avoid using it whenever possible.



**Reference**
1. SQL indexes for "not equal" searches - [here](https://stackoverflow.com/questions/2864267/sql-indexes-for-not-equal-searches)
2. PostgreSQL 8.0.26 Documentation Chapter 11. Indexes - [here](https://www.postgresql.org/docs/8.0/indexes-partial.html)