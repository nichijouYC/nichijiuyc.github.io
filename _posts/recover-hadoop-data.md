---
title: hadoop节点数据丢失恢复
date: 2016-09-16 18:51:36
tags:
---


### hadoop节点数据丢失恢复



#### 现有环境

- 正常运行的hadoop集群三台

        1. ubuntu1:namenode,datanode

        2. ubuntu2:datanode

        3. ubuntu3:datanode

- 设置dfs.replication=2

<!-- more -->

#### datanode数据丢失恢复

1. 上传文件(`./hadoop fs -put ../LICENSE.txt /`),查看文件存储的节点

    - LICENSE.txt 存在 ubuntu1,ubuntu3

    - README.txt 存在 ubuntu1,ubuntu2

2. 查看文件(`./hadoop fs -cat hdfs://ubuntu1:9000/README.txt`),可以正常查询

3. 将其中ubuntu2节点的datanode进程kill掉,模拟datanode节点挂掉,数据丢失

    - 默认超时时间为10.5分钟,超过超时时间后可以在50070页面上看到datanode挂掉

4. 查看文件(`./hadoop fs -cat hdfs://ubuntu1:9000/README.txt`),可以正常查询

5. 查看此时文件存储的节点

    - LICENSE.txt 存在 ubuntu1,ubuntu3

    - README.txt 存在 ubuntu1,ubuntu3



#### 原因探究:

1. datanode每隔固定时间会向namenode发送心跳包,报告datanode状态和数据块

2. 当datanode挂掉后,namenode根据datanode最后一次发送心跳的时间,超过阈值后,就认为这个datanode挂掉了

3. namenode确认datanode挂掉后,会确定这个节点上的block,查看他的副本数,如果低于复制因子,会从副本datanode复制这个数据块到其他datanode

4. block复制完成后,集群数据副本数量正常.若挂掉的datanode恢复后,数据副本数量超过复制因子,集群会随机删除多余副本



#### namenode数据丢失恢复



1. 将ubuntu1的namenode进程kill掉,并且将hadoop数据目录下的/dfs/name里的文件删除,模拟namenode数据丢失

2. 查看文件(`./hadoop fs -cat hdfs://ubuntu1:9000/README.txt`),提示错误,拒绝连接

3. 重启namenode,发现namenode无法启动

4. 将secondarynamenode节点中的/dfs/namesecondary内容(checkpoint)复制到namenode节点中的/dfs/name文件夹下

5. 执行`bin/hadoop namenode -importCheckpoint`,使namenode读取checkpoint文件

6. 再重新启动namenode,启动成功

7. 查看文件(`./hadoop fs -cat hdfs://ubuntu1:9000/README.txt`),可以正常查询



#### 原因探究:

1. namenode会把文件系统的变化保存到日志文件edits中,namenode启动时,会从fsimage中读取HDFS的状态,并且把edits文件中的操作应用到fsimage,合并后创建一个新的edits文件来记录文件系统的变化

2. namenode只有在启动时才会合并edits和fsimage,secondarynamenode会定期合并fsimage和edits日志(默认配置1个小时),并且保存最后一次checkpoint结果

3. secondarynamenode保存的结果和namenode保存结果格式一样,所以当namenode数据丢失时,可以拷贝secondarynamenode的数据,恢复namenode,但是是一小时之前的结果


