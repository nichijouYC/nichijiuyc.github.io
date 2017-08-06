---
title: ubuntu安装spark
date: 2016-06-13 20:40:09
tags:
---


### ubuntu安装spark

<!-- more -->

0. 必须先安装ssh java hadoop
1. 下载spark包
2. 将安装包scp到某一节点并解压 
`tar zxf spark-2.0.0-bin-hadoop2.6.tgz`
3. 移动到home目录
`mv spark-2.0.0-bin-hadoop2.6 /home/fyc/spark`
4. 修改/etc/profile文件

    ```
    export SPARK_HOME=/home/fyc/spark
    export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
    ```

5. 保存profile
`source /etc/profile`
6. 修改spark配置文件
    - （spark-env.sh）
    ```
    export SPARK_MASTER_IP=10.10.101.34
    export SPARK_WORKER_MEMORY=64g
    ```

    - (spark-config.sh)
    ```
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
    ```

    - (slave)
    ```
    slave1
    slave2
    ```
7. 将SPARK_HOME整个文件夹拷贝到剩余所有节点
8. spark相关命令
  - 启动 `start-all.sh`
  - 停止 `stop-all.sh`
  - WEB UI `http://10.10.101.34:8080`
9. 启动sparksql thriftserver jdbc服务
    1. 先启动spark和hive metastore
    2. 启动sparksql`/home/fyc/spark/sbin/start-thriftserver.sh --master spark://10.10.101.34:7077 --executor-memory 8g  --executor-cores 8 --total-executor-cores 72 --conf spark.sql.parquet.mergeSchema=true --conf spark.speculation=true`
