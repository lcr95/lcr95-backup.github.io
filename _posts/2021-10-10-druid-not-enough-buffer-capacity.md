---
layout: post
title: "Druid Not Enough Buffer Capacity"
subtitle: ""
date: 2021-10-10
author: "ChenRiang"
header-style: text
tags:
    - Druid
---



When running a groupby query with huge aggregation process and follow by sub select query in SQL, I bump into following exception



> Not enough capacity for even one row! Need[xx] but have[0]



**Solution**

Configure broker node with higher buffer size and number mergeBuffer will solve the exception.

```
druid.processing.buffer.sizeBytes=500MiB
druid.processing.numMergeBuffers=4
```



