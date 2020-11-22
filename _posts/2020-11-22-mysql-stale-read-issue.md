---
layout:     post
title:      "MySQL Cluster Stale Read Issue"
subtitle:   ""
date:       2020-11-22 12:00:00
author:     "ChenRiang"
header-style: text
catalog: true
tags:
    - MySQL
---

In my recent work, I have a requirement to increase the throughput of MySQL which do heavy READ operation and light WRITE operation. 
We all know that single-node MySQL will defiantly not able to achieve it and running it in cluster mode(Master-Slave) would probably 
could help, as we could spread out the workload by scaling up the number of slave instance.

# The Problem
I could get a pretty good result by scaling up the slave instance. <br/> 

However, I and the team bump into a situation that 
we have to ensure whatever we `INSERT` into MySQL would be found in the subsequent `SELECT` statement. The data replication in 
MySQL slave is asynchronous, and the "lagging" in between caused data inaccuracy. 

**Stale reads** - read operation that fetches an incorrect value from a source that has not synchronized an update operation to the value


# The Solution
We have to sacrifice some throughput to ensure the result accuracy. 

There are 2 builtin method by MySQL that could help us to solve this issue:

`SHOW MASTER STATUS` - provide the binlog file name and data position. <br/>
`MASTER_POS_WAIT(log_name, log_position)` - block until the replica has read and applied all the updates to the specified position 

<br/>
Steps:

1. `INSERT` data to Master Node
2. `SHOW MASTER STATUS` to query out the Master Node's binlog and position
3. `SELECT MASTER_POS_WAIT` on the target slave node.
4. `SELECT` data from slave node.

{% include image.html src="post-mysql-stale-read.png" data="group" title="Steps to fix stale read" %}
