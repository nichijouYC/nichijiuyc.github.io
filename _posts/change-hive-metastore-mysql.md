---
title: 修改hive metasore为mysql
date: 2016-10-15 23:07:41
tags:
---


### 修改hive metasore为mysql


- hive读取的数据默认在当前路径下的metastore文件中(.derby文件)
- 如果需要从不同路径访问hive或者远程访问,则需要重新配置hive metastore 存储在mysql中
- 配置完成后,需要先开metastore服务(`nohup hive --service metastore -p 9083 &`),才能使用hive

<!-- more -->
- 步骤
    1. 拷贝mysq的驱动到`$hive_home/lib`目录下
    2. 在mysql中新增hive用户,和hive数据库
    3. 修改`hive-site.xml`
    ```
    <?xml version="1.0"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://localhost:9083</value>
        <description>Thrift URI for the remote metastore. Used by metastore client to connect to remote metastore.</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
        <description>JDBC connect string for a JDBC metastore</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
        <description>Driver class name for a JDBC metastore</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>hive</value>
        <description>username to use against metastore database</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>hive</value>
        <description>password to use against metastore database</description>
    </property>
    </configuration>
    ```

