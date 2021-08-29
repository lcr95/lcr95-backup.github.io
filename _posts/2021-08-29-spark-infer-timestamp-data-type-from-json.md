---
layout: post
title: "Spark Infer Timestamp data type from JSON"
subtitle: ""
date: 2021-08-29
author: "ChenRiang"
header-style: text
tags:
    - Spark
    - Python
---



Apache Spark provides a feature to infer data schema based on the incoming data. However, in Spark version 3.1.2, it wrongly interprets the Timestamp field as String data type.



```python
import pyspark
spark = pyspark.sql.SparkSession.builder.appName("MyApp").getOrCreate()

line = '{"myTimestamp" : "2017-09-30 04:53:39.412496Z"}'
rdd = spark.sparkContext.parallelize([line])
(
    spark.read
    .option("timestampFormat", "yyyy-MM-dd HH:mm:ss.SSSSSS'Z'")
    .json(path=rdd)
)
```

Result:

```
DataFrame[myTimestamp: string]
```

<br/>

**Solution**

After some googling, I found out that there is an option `inferTimestamp` have to be enabled to allow Spark recognize timestamp data. This option is disabled by default in Spark 3.0.1 and above.



```python
line = '{"myTimestamp" : "2017-09-30 04:53:39.412496Z"}'
rdd = spark.sparkContext.parallelize([line])
(
    spark.read
    .option("inferTimestamp", "true")
    .option("timestampFormat", "yyyy-MM-dd HH:mm:ss.SSSSSS'Z'")
    .json(path=rdd)
)
```

Result:

```
DataFrame[myTimestamp: timestamp]
```



** Note: `inferTimestamp` was disabled intentionally due to performance issue. It is recommended to provide schema and avoid schema generation on the fly when ingesting data.



<br/>

**Reference**

1. Migration Guide: SQL, Datasets and DataFrame - [Documentation](https://spark.apache.org/docs/latest/sql-migration-guide.html#:~:text=Set%20JSON%20option-,inferTimestamp,-to%20false%20to)
2. Interpret timestamp fields in Spark while reading json (timestampFormat) - [SPARK-26325](https://issues.apache.org/jira/browse/SPARK-26325)
3. Spark 3.0 json load performance is unacceptable in comparison of Spark 2.4 - [SPARK-32130](https://issues.apache.org/jira/browse/SPARK-32130)

