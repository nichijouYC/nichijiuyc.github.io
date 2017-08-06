---
title: Docker技术入门与实战 笔记
date: 2016-08-03 21:03:36
tags:
---


### Docker技术入门与实战 笔记


#### Docker 简介

- Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上

<!-- more -->
#### Docker 核心概念

1. 镜像（Image）
    - Docker镜像类似于虚拟机镜像，相当于面向Docker的只读模板
2. 容器（Container）
    - Docker容器是根据镜像创建的应用运行实例，可以启动、开始、停止、删除。对容器的修改写入容器的文件系统中，并不会写入Docker镜像中
3. 仓库（Repository）
    - Docker仓库是存放镜像文件的地方。注册服务器时存放仓库的地方。某个仓库集中存放某一类镜像，包括多个镜像文件，通过tag区分，比如不同版本的镜像

#### Docker安装

- 参见官网[安装教程](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
- 安装完成后运行 `sudo service docker start` 和 `sudo docker run hello-world`，验证是否安装成功 

#### 镜像

1. 获取镜像 `sudo docker pull ubuntu` 不指定tag默认下载latest版本的，或者指定tag下载指定版本 `docker pull ubuntu:14:04`
2. 获取镜像详细信息 `sudo docker inspect ubuntu`
3. 查看镜像 `sudo docker images`
4. 查看所有容器 `sudo docker ps -a`
5. 创建自己的新的镜像
    1. 基于已有镜像的容器创建
        1. 先运行一个镜像 `sudo docker run -it ubuntu /bin/bash`
        2. 做一些修改 比如增加一个文件 `touch a`
        3. 退出容器 `exit` 并记住容器id
        3. 根据这个容器创建一个新的镜像 `sudo docker commit -m "message" -a "author" 4acc test`
        4. 查看镜像 `sudo docker images` 会有一个新的镜像（test） 
    2. 基于本地模板导入
    3. 基于DockerFile创建
6. 保存镜像（将镜像保存到本地文件） `sudo docker save -o ubuntu.tar ubuntu:14.04`
7. 导入镜像(从本地文件导入镜像)  `sudo docker load --input ubuntu.tar`
8. 上传镜像 `sudo docker push test:latest`
9. 给镜像打标签 `sudo docker tag test:lastest ubuntu:14.04` 
10. 删除镜像 `sudo docker rmi ubuntu`

#### 容器

1. 新建容器 `sudo docker create ubuntu:latest` 新建的容器是停止状态的
2. 启动容器 `sudo docker start 4acc`
3. 新建并且启动容器 `sudo docker run -it ubuntu /bin/bash`   
    -t 指分配一个伪终端   
    -i 指保持打开容器的标准输入 exit后 容器处于停止状态
4. 后台运行容器 `sudo docker run -d -P ubuntu /bin/echo "hello world"`  
    -d 后台运行   
    -P 转发容器所有端口到主机端口，可以通过主机ip：端口访问  
    如果不加-P 宿主机外无法访问，宿主机可以通过容器ip:端口号访问 另外可以指定ip映射  -p 5000:5000 
5. 获取后台容器输出`docker logs 4acc`
6. 停止容器 `sudo docker stop 4acc` 可以通过start命令来重新启动
7. 进入容器 `sudo docker exec -it 4acc /bin/bash` 容器必须是运行状态
8. 删除容器 `sudo docker rm 4acc`
9. 导出容器 `sudo docker export 4acc > ubuntu.tar`
10. 导入容器（成为镜像） `cat ubuntu.tar | sudo docker import - ubuntu:latest` 
    - 与导入镜像的区别： 容器快照会丢弃所有的历史记录和元数据信息（比如标签） 镜像文件会保存完整记录，所以体积比容器文件大很多 
11. 获取容器详细信息 `sudo docker inspect 4acc` 包括ip地址等

#### 仓库

1. 搜索镜像 `sudo docker search hadoop`
2. 下载指定注册服务器的镜像 `sudo docker pull dl.dockerpool.com:5000/tomcat`
3. 创建和使用私有仓库
    1. 创建私有仓库 `sudo docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry`  
       在本地启动一个registry容器 监听5000端口 镜像文件默认存在容器的/tmp/registry中 -v 可以将镜像文件存放在本地目录上
    2. 上传镜像(私有仓库地址为10.0.2.2:5000) `sudo docker push 10.0.2.2:5000/test`
    3. 下载镜像 `sudo docker pull 10.0.2.2:5000/test`


#### 使用DaoCloud加速器

- 在国内获取镜像时（docker pull），由于网络问题，下载镜像文件非常慢。我们可以使用DaoCloud的加速器工具加快下载镜像的速度。DaoCloud通过智能路由和缓存机制，极大提升了国内网络访问 Docker Hub的速度
- 使用步骤（[官网文档](https://www.daocloud.io/mirror.html#accelerator-doc)）
    1. 运行 `curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://404114fe.m.daocloud.io`
    2. 重启docker服务 `sudo service docker restart` 即可

#### 数据管理

- 数据卷
    1. 数据卷是一个可以供容器使用的特殊文件目录，可以在不同容器之间共用共享
    2. 对数据卷的修改马上生效
    3. 对数据卷更新不会影响镜像
    4. 卷会一直存在，直到没有容器使用

1. 在容器中创建数据卷 `sudo docker run -v /webapp ubuntu`
2. 挂载本地主机目录或者文件作为数据卷 `sudo docker run -d -P tomcat -v /src/webapp:/opt/webapp`

- 数据卷容器
    1. 如果需要在容器间共享一些持续更新数据，可以使用数据卷容器
    2. 数据卷容器是一个普通的容器，其他多个容器可以挂载这个数据卷容器获取数据，数据共享
    3. 数据卷容器也可以挂载本地目录，实现主机/容器之间的数据共享

1. 创建数据卷容器 `sudo docker run -it -v /dbdata  --name dbdata ubuntu` --name 指定容器名字
2. 在别的容器中挂载数据卷容器  `sudo docker run -it --volumes-from dbdata --name db1 ubuntu`  (数据卷容器不需要在运行状态)

#### 容器互联

- 有时我们会遇到容器间需要通信的时候，比如一个容器为tomcat，另一个容器为mysql，tomcat容器需要链接mysql容器，有两种方式实现容器间通信

- 方式一：每个容器都将端口映射到主机，通过主机进行容器互联
- 方式二：使用link命令


