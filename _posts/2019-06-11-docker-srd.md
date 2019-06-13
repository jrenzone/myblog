---
layout:     post
title:      "基于Docker的采集服务部署"
subtitle:   "调度服务加采集服务部署"
date:       2019-06-10 12:00:00
author:     "Jren"
header-img: "img"
tags:
    - Docker
    - 运维
---
> 这里记录一下公司里的一套数据采集系统的部署过程

## 简单介绍

### 系统架构
结构很简单，就是加了 1个平台下挂着两个干活的服务。
*   任务调度平台-[XXL-JOB](http://www.xuxueli.com/xxl-job/#/)
*   具体的两套采集服务
### 需要做的
*   Docker部署mysql
*   Docker部署xxl-job
*   Jar包部署采集服务

## 部署
### 创建桥接网络
1.  服务器上创建一个名为my-bridge的桥接网络`docker network create -d bridge my-bridge`
2.  查看下当前docker服务中网络：`docker network ls`
### Mysql
1.  拉取Mysql:5.6镜像。
2.  创建mysql目录：`sudo mkdir /opt/app/mysql`
3.  进入到mysql目录，执行docker命令：`docker run -p 3306:3306 --name mymysql --network my-bridge -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6`
4.  这个时候可能会报错，比如：`docker: Error response from daemon: driver failed programming external connectivity on endpoint mymysql (0ae2ab8e15cb6a7asss4c8881ab552e7349b6e2a38dcdf70eaea222b0dc41e): Error starting userland proxy: mkdir /port/tcp:0.0.0.0:3306:tcp:172.19.0.2:3306: input/output error.`<br>解决方法就是重启docker服务，我本地使用的是 win10 pro+ubuntu子系统+docker desktop,我直接重启的docker desktop之后，直接`docker start mymysql` 即可。
### xxl-job
1.  拉取xxl-job-admin:2.0.2镜像
2.  docker命令:`docker run -e PARAMS="--spring.datasource.url=jdbc:mysql://mymysql:3306/xxl-job?Unicode=true&characterEncoding=UTF-8&useSSL=false" -p 8081:8080 -v /tmp:/data/applogs --name xxl-job-admin --network my-bridge -d xuxueli/xxl-job-admin:2.0.2` <br> 备注： xxl-job-admin 默认的docker启动方式是支持参数输入的，可惜，使用起来总是有问题，用户密码不对..所以这里也不纠结，可以下载源码本地编译BUILD成镜像再启动即可。
3.  
### 采集服务
1.  采集服务当前还是使用的java -jar 的方式，感觉没什么可说的。

### 待续

老大要准备搞集群部署，上云，还需要搭建一个网关...后面继续