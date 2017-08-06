---
title: ubuntu安装hbase
date: 2016-06-22 19:03:51
tags:
---
### ubuntu安装hbase

<!-- more -->
0. 必须先安装ssh java hadoop
1. 下载hbase包
2. 将安装包scp到某一节点并解压  `tar zxf hbase-0.98.19-hadoop2-bin.tar.gz`
3. 移动到home目录 `mv hbase-0.98.19-hadoop2 /home/fyc/hbase`
4. 修改hbase配置文件
    - (hbase-site.xml)
    ```
        <property>
                <name>hbase.rootdir</name>
                 <!-- 对应于hdfs中配置 -->
                <value>hdfs://hty-compute:9000/hbase</value>
        </property>
        <property>
                 <name>hbase.cluster.distributed</name>
                 <value>true</value>
        </property>
        <property>
                <name>hbase.master</name>
                <value>hty-compute:60000</value>
        </property>
         <property>
                 <name>hbase.master.port</name>
                 <value>60000</value>
                <description>The port master should bind to.</description>
         </property>
    ```
    - (hbase-env.sh)
    ```
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/jre
    export HBASE_CLASSPATH=/root/fyc/hadoop-2.5.2/etc/hadoop
    export HBASE_HOME=/home/fyc/hbase 
    export HBASE_MANAGES_ZK=true
    ```
    - (regionservers)
    ```
    10.100.100.214
    10.100.100.215
    10.100.100.216
    ```

5. 将hbase整个文件夹拷贝到剩余所有节点
6. hbase相关命令
- 启动 `start-hbase.sh`
- 停止 `stop-hbase.sh`
- shell `hbase shell`
- WEB UI `http://10.10.101.34:60010`

