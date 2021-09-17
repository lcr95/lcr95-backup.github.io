---
layout: post
title: "Connect Delta Lake with JDBC"
subtitle: ""
date: 2021-09-17
author: "ChenRiang"
header-style: text
tags:
    - DeltaLake
    - Spark
    - Java
---



In previous blog [post]({% post_url 2021-09-09-delta-lake-the-next-gen-data-lake %}), we walk through some basic CRUD operations on Delta Lake. However, if you're a Java application developer, you might just want to focus on the SQL query logic without having to worry about the details of Spark.

> "Can I access Delta Lake with JDBC? "

"**Yes**.  Use Spark Thrift Server (STS)"  



# Spark Thrift Server (STS)

STS is basically an [Apache Hive Server2](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2) that enable JDBC/ODBC client to execute queries remotely to  retrieve results. The differences between STS and hiveSever2 is that instead of  submitting the SQL queries as hive map reduce job STS will use Spark SQL engine. With STS, you will able to leverage the full spark capabilities to perform the queries. 



# Start/Stop Server

Run the following command in the Spark distribution folder (SPARK_HOME).

Start server with Spark local mode: 

```bash
sbin/start-thriftserver.sh \
  --conf spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension \
  --conf spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog \
  --packages 'io.delta:delta-core_2.12:1.0.0'
```



Start server on existing Spark cluster:

```bash
sbin/start-thriftserver.sh \
  --conf spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension \
  --conf spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog \
  --packages 'io.delta:delta-core_2.12:1.0.0' \ 
  --master spark://<spark host>:<spark port> 
```



Start server with S3 aceess credential:

```bash
sbin/start-thriftserver.sh \
  --conf spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension \
  --conf spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog \
  --conf spark.hadoop.fs.s3a.access.key=<s3 access key> \
  --conf spark.hadoop.fs.s3a.secret.key=<s3 secret key> \
  --packages 'io.delta:delta-core_2.12:1.0.0, org.apache.hadoop:hadoop-aws:3.3.1' \ 
  --master spark://<spark host>:<spark port> 
```



Stop server:

```bash
sbin/stop-thriftserver.sh
```



# Connect with JDBC

To test the JDBC connection we will use the `beeline` tools that package in Spark's `bin` folder.

 ```bash
 bin/beeline
 ```



Connect to STS :

```bash
beeline> !connect jdbc:hive2://localhost:10000
```

 ** Beeline will prompt for username and password. In non-secure mode, simply enter the username on your machine and a blank password. For secure mode, please follow the instructions given in the [beeline documentation](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients).



Once connected you can simply issue the SQL queries. 

{% include image.html src="jdbc-query.png" data="group" title="" %}





# Conclusion 

In this blog post, we looked into using Spark thrift server to query Delta lake using JDBC.

  



**Reference**

1. Disrtibuted SQL Engine - [link](https://spark.apache.org/docs/latest/sql-distributed-sql-engine.html)
2. Thrift JDBC/ODBC Server — Spark Thrift Server (STS) - [link](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/spark-sql-thrift-server.html) 
3. HiveServer2 Clients - [link](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-PythonClient)
