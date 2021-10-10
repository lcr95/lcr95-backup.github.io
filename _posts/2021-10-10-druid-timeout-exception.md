---
layout: post
title: "Druid Timeout Exception"
subtitle: "org.apache.druid.query.QueryTimeoutException"
date: 2021-10-10
author: "ChenRiang"
header-style: text
tags:
    - Druid
---



I hit timeout exception from Druid when I performing a big query that require long time to process. 

```
org.apache.druid.query.QueryTimeoutException: url[

[http://xx.xx.xx.xx:8088/druid/v2/]](http://xx.xx.xx.xx:8088/druid/v2/])

timed out

at org.apache.druid.client.JsonParserIterator.timeoutQuery(JsonParserIterator.java:204)

at org.apache.druid.client.JsonParserIterator.next(JsonParserIterator.java:119)

at org.apache.druid.java.util.common.guava.BaseSequence.makeYielder(BaseSequence.java:90)

at org.apache.druid.java.util.common.guava.BaseSequence.access$000(BaseSequence.java:27)

at org.apache.druid.java.util.common.guava.BaseSequence$1.next(BaseSequence.java:114)

at org.apache.druid.java.util.common.guava.MergeSequence.makeYielder(MergeSequence.java:131)

at org.apache.druid.java.util.common.guava.MergeSequence.access$000(MergeSequence.java:32)

at org.apache.druid.java.util.common.guava.MergeSequence$2.next(MergeSequence.java:173)

at org.apache.druid.java.util.common.guava.WrappingYielder$1.get(WrappingYielder.java:53)

at org.apache.druid.java.util.common.guava.WrappingYielder$1.get(WrappingYielder.java:49)

at org.apache.druid.java.util.common.guava.SequenceWrapper.wrap(SequenceWrapper.java:55)

at org.apache.druid.java.util.common.guava.WrappingYielder.next(WrappingYielder.java:48)

at org.apache.druid.java.util.common.guava.MergeSequence.makeYielder(MergeSequence.java:131)

at org.apache.druid.java.util.common.guava.MergeSequence.access$000(MergeSequence.java:32)

at org.apache.druid.java.util.common.guava.MergeSequence$2.next(MergeSequence.java:173)

at org.apache.druid.common.guava.CombiningSequence$1.next(CombiningSequence.java:139)

at org.apache.druid.java.util.common.guava.WrappingYielder$1.get(WrappingYielder.java:53)

at org.apache.druid.java.util.common.guava.WrappingYielder$1.get(WrappingYielder.java:49)

at org.apache.druid.java.util.common.guava.SequenceWrapper.wrap(SequenceWrapper.java:55)

at org.apache.druid.java.util.common.guava.WrappingYielder.next(WrappingYielder.java:48)

at org.apache.druid.java.util.common.guava.WrappingYielder$1.get(WrappingYielder.java:53)

at org.apache.druid.java.util.common.guava.WrappingYielder$1.get(WrappingYielder.java:49)

at org.apache.druid.query.CPUTimeMetricQueryRunner$1.wrap(CPUTimeMetricQueryRunner.java:78)

at org.apache.druid.java.util.common.guava.WrappingYielder.next(WrappingYielder.java:48)

at org.apache.druid.java.util.common.guava.WrappingYielder$1.get(WrappingYielder.java:53)

at org.apache.druid.java.util.common.guava.WrappingYielder$1.get(WrappingYielder.java:49)

at org.apache.druid.java.util.common.guava.SequenceWrapper.wrap(SequenceWrapper.java:55)

at org.apache.druid.java.util.common.guava.WrappingYielder.next(WrappingYielder.java:48)

at org.apache.druid.sql.avatica.DruidStatement.nextFrame(DruidStatement.java:278)

at org.apache.druid.sql.avatica.DruidMeta.fetch(DruidMeta.java:238)

at org.apache.calcite.avatica.remote.LocalService.apply(LocalService.java:239)

at org.apache.calcite.avatica.remote.Service$FetchRequest.accept(Service.java:1370)

at org.apache.calcite.avatica.remote.Service$FetchRequest.accept(Service.java:1337)

at org.apache.calcite.avatica.remote.AbstractHandler.apply(AbstractHandler.java:94)

at org.apache.calcite.avatica.remote.JsonHandler.apply(JsonHandler.java:52)

at org.apache.calcite.avatica.server.AvaticaJsonHandler.handle(AvaticaJsonHandler.java:133)

at org.apache.druid.sql.avatica.DruidAvaticaHandler.handle(DruidAvaticaHandler.java:59)

at org.eclipse.jetty.server.handler.HandlerList.handle(HandlerList.java:59)

at org.eclipse.jetty.server.handler.StatisticsHandler.handle(StatisticsHandler.java:179)

at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:127)

at org.eclipse.jetty.server.Server.handle(Server.java:516)

at org.eclipse.jetty.server.HttpChannel.lambda$handle$1(HttpChannel.java:388)

at org.eclipse.jetty.server.HttpChannel.dispatch(HttpChannel.java:633)

at org.eclipse.jetty.server.HttpChannel.handle(HttpChannel.java:380)

at org.eclipse.jetty.server.HttpConnection.onFillable(HttpConnection.java:277)

at org.eclipse.jetty.io.AbstractConnection$ReadCallback.succeeded(AbstractConnection.java:311)

at org.eclipse.jetty.io.FillInterest.fillable(FillInterest.java:105)

at org.eclipse.jetty.io.ChannelEndPoint$1.run(ChannelEndPoint.java:104)

at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.runTask(EatWhatYouKill.java:336)

at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.doProduce(EatWhatYouKill.java:313)

at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.tryProduce(EatWhatYouKill.java:171)

at org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.run(EatWhatYouKill.java:129)

at org.eclipse.jetty.util.thread.ReservedThreadExecutor$ReservedThread.run(ReservedThreadExecutor.java:383)

at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:882)

at org.eclipse.jetty.util.thread.QueuedThreadPool$Runner.run(QueuedThreadPool.java:1036)

at java.lang.Thread.run(Thread.java:748)

Suppressed: com.fasterxml.jackson.databind.JsonMappingException: Query[ce2cb9ab-ab5b-42f8-b348-e58b94a2dbf3] url[

[http://xx.xx.xx.xx:8088/druid/v2/]](http://xx.xx.xx.xx:8088/druid/v2/])

timed out. (through reference chain: java.lang.Object[][2])
```



<br/>

**Solution**

Set the following properties in broker nodes to have long time out.

```
druid.server.http.defaultQueryTimeout=3000000
```

