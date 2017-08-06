---
title: spark宽依赖和窄依赖
date: 2016-12-21 19:10:43
tags:
---


### spark宽依赖和窄依赖



- 在Spark中，RDD之间的依赖关系有两种，宽依赖和窄依赖。RDD的高效与DAG图有着很大的关系，在DAG调度中需要对计算过程划分stage，而划分依据就是RDD之间的依赖关系。



- Spark中每次Action和Transaction都会生成一个新的RDD。子RDD会依赖于父RDD。不同类型的Transaction，会产生不同的依赖关系。

<!-- more -->

#### 窄依赖

1. 窄依赖是指一个父RDD只对应于一个子RDD

2. 窄依赖的函数有map、filter、uoion等等



#### 宽依赖

1. 宽依赖是指一个父RDD对应于多个子RDD

2. 宽依赖的函数有groupByKey、join等等



#### 两者区别

1. 宽依赖往往会引发shuffle操作，需要将一个父RDD的结果传到不同的子RDD中，涉及到多个节点的数据传输（shuffle），影响效率。而窄依赖中，每个父RDD只会对应于一个子RDD，通常在同一个节点中，效率比较高。所以尽量使用窄依赖而不是宽依赖



2. 在RDD分区丢失时（比如某个节点发生故障时），spark会重新计算数据。对于窄依赖，只需要重算他的父RDD即可。而对于宽依赖，需要重新计算所有父RDD，会造成很多冗余计算。



#### Application、Job、Stage、Task划分依据



1. Application：一个提交的spark的程序即为一个Application

2. Job：一个Application中，每个Action算子会划分成一个Job

3. Stage：一个Job中，每个宽依赖操作（shuffle）会划分成一个Stage

4. Task：Task是Spark的最小执行单位。RDD中每个分区（partation）都对应一个Task，一般每个文件Block对应于一个Task


