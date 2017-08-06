---
title: ubuntu安装hadoop(完全分布式)
date: 2016-06-03 10:37:15
tags:
---

### ubuntu安装hadoop(完全分布式)

<!-- more -->
1. 安装java环境
`sudo apt-get install openjdk-7-jdk`
2. 配置ssh（每个节点相互ssh免密登陆）
`sudo apt-get install ssh`
`ssh-keygen -t rsa`
`ssh-copy-id -i ~/.ssh/id_rsa.pub remote-host`
`ssh localhost 验证`
3. 修改`/etc/host` 加入每个节点的ip和hostname
4. 修改/etc/profile文件
    ```
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64  
    export PATH=$PATH:$JAVA_HOME/bin  
    export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
    export HADOOP_HOME=/home/fyc/hadoop-2.6.2  
    export PATH=$PATH:$HADOOP_HOME/bin  
    export PATH=$PATH:$HADOOP_HOME/sbin  
    export HADOOP_MAPRED_HOME=$HADOOP_HOME
    export HADOOP_COMMON_HOME=$HADOOP_HOME  
    export HADOOP_HDFS_HOME=$HADOOP_HOME  
    export YARN_HOME=$HADOOP_HOME  
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
    ```
5. 保存profile
`source /etc/profile`
6. 下载hadoop包
7. 将安装包scp到某一节点并解压
`sudo tar xzf hadoop-2.6.2.tar.gz`
8. 移动到home目录
`sudo mv hadoop-2.6.2 /home/fyc`
9. 查看java安装路径
`update-alternatives --config java`
`/usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java`
10. 修改hadoop配置文件  
    - （core-site.xml）
    ```
        <property>
                <name>fs.default.name</name>
                <value>hdfs://10.10.101.34:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/home/fyc/hadoop-2.6.2/tmp</value>
                <description>Abasefor other temporary directories.</description>
         </property>
    ```
    - (hdfs-site.xml)
    ```
        <property>
                <name>dfs.replication</name>
                <value>3</value>
        </property>
        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
        <property>
                <name>dfs.http.address</name>
                <value>10.10.101.34:50070</value>
        </property>
        <property>
                <name>dfs.balance.bandwidthPerSec</name>
                <value>1048576</value>
        </property>
        <property>
                <name>fs.hdfs.impl</name>
                <value>org.apache.hadoop.hdfs.DistributedFileSystem</value>
        </property>
        <property>  
                <name>dfs.namenode.secondary.http-address</name>  
                <value>10.10.101.34:50090</value>  
        </property>  


    ```
    - (mapred-site.xml)
    ```
        <property>
                <name>mapred.job.tracker</name>
                <value>10.10.101.34:9001</value>
        </property>
    ```
    - (hadoop-env.sh)
    ```
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/jre
    ```
    - (slave)
    ```
    slave1
    slave2
    ```

    - (masters)
    ```
    slave1
    ```
11. 将HADOOP_HOME整个文件夹拷贝到剩余所有节点
12. hadoop相关命令
- 初始化hadoop `hadoop namenode -format`
- 启动 `start-all.sh`
- 停止 `stop-all.sh`
- 上传文件 `hadoop fs -put ...`
- 查看文件 `hadoop fs -ls /`
- WEB UI `http://10.10.101.34:50070`

