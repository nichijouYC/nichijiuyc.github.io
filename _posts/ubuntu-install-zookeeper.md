---
title: ubuntu安装zookeeper
date: 2016-06-13 20:50:36
tags:
---

### ubuntu安装zookeeper
<!-- more -->
0. 必须先安装ssh java
1. 下载zookeeper包
2. 将安装包scp到某一节点并解压 
`tar zxf zookeeper-3.3.4.tar.gz`
3. 移动到home目录
`mv zookeeper-3.3.4 /home/fyc/zookeeper`
4. 修改配置文件
    - (zoo.cfg)
    ```
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/root/fyc/zookeeper/tmp
    clientPort=2181
    server.1=10.10.101.34:2888:3888  
    server.2=10.10.101.208:2888:3888  
    server.3=10.10.101.209:2888:3888 
    ```
5. 将zookeeper整个文件夹拷贝到剩余所有节点
6. 在每个节点上创建myid文件
    - 在配置的dataDir指定的目录下面，创建一个myid文件，里面内容为一个数字，用来标识当前主机,zoo.cfg文件中配置的server.X中X为什么数字，则myid文件中就输入这个数字
    - `echo "1" > /home/fyc/zookeeper/myid  `
    
7. 在每个节点上启动zookeeper
`bin/zkServer.sh start `
8. 查看zookeeper 状态
`bin/zkServer.sh status`
