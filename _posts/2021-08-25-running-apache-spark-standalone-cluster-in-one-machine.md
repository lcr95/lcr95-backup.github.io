---
layout: post
title: "Running Apache Spark standalone cluster in one machine"
subtitle: ""
date: 2021-08-25
author: "ChenRiang"
header-style: text
tags:
    - Spark
---



Apache Spark is a complex framework that provide parallelized in-memory data processing. Often time during development, we just want to focus on creating the business without having to worried about the infrastructure. In this blog post, we will look into how we can run Apache Spark cluster in single machine.



1. Download Apache Spark from [here](https://spark.apache.org/downloads.html) and unzip it.

2. In Spark directory, run following command to start Spark Master.

   ```bash
   ./sbin/start-master.sh
   ```

   

3. Start Spark Worker.

   ```bash
   ./bin/spark-class org.apache.spark.deploy.worker.Worker spark://`hostname`:7077
   ```

   

4. Submit spark job

   ```bash
   ./bin/spark-submit \
      --class <your main class> \
      --master spark://`hostname`:7077 \
      <your jar file> 
   ```

   

