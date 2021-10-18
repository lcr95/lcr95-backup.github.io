---
layout: post
title: "Druid Not Enough Capacity for Primary Segment"
subtitle: ""
date: 2021-10-18
author: "ChenRiang"
header-style: text
tags:
    - Druid
---



The error below being thrown by Historical node when the Druid cluster loading the data segment.

>  2021-10-01T01:51:13,538 WARN [Coordinator-Exec--0] org.apache.druid.server.coordinator.rules.LoadRule - No available [_default_tier] servers or node capacity to assign primary segment



**Solution**

This error happened because the Druid cluster has only one Histroical node and the segment is configured to create two replicas (default) of each segment. To solve the error there is two ways.



<u>Solution 1</u>  

The easiest solution.  Spin up few more historical nodes according to the configuration needs. 

<br/>

<u>Solution 2</u> 

If spinning more node server is not possible. We can configure the segment to only create lesser replica for each segment. 

1. Login to the Druid UI console and click on "**Datasources**".

2. Look your datasource and click **edit** button.

3. Click "**Edit retention rules**"

   {% include image.html src="edit-rule.png" data="group" title="" %}

   

   In my case, I have only have the default rule and below is the configuration 

   {% include image.html src="default-rule.png" data="group" title="" %}

   

4. Decrease the **number of replicants** according to your cluster. In my case, I will set it to 1.

   {% include image.html src="segment-repliace-one.png" data="group" title="" %}

   

5. Click "**Next**"

6. Describe your changes and hit "**Save**".

   {% include image.html src="desc-changes.png" data="group" title="" %}

   

7. Done.

<br/>

In case, after you apply either one of the solution mentioned above and the error still happening. There is another configuration in Historical Node  you might want to check it out:

> druid.segmentCache.locations= [{\"path\":\"/druid/data/segment-cache\",**\"maxSize\":100737418240**}]



Make sure the `maxSize` is enough for the segment. Click [here](https://druid.apache.org/docs/latest/configuration/index.html#storing-segments) to understand more about the configuration. 





**Reference** 

1. Github Issue ticket - [link](https://github.com/apache/druid/issues/5808#issuecomment-393167532)

