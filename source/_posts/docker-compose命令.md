---
title: docker-compose命令
typora-root-url: ../
date: 2022-11-13 22:37:53
tags: 
 - docker
 - docker compose
categories: 
 - docker
---



## 命令


查看版本号
```bash
docker compose version
```


查看帮助

```bash
docker compose -h
```


启动所有docker-compose服务

```bash
docker compose up
```


启动所有docker-compose服务并后台运行

```bash
docker compose up -d  
```


停止并删除容器、网络、卷、镜像。

```bash
docker compose down   
```


进入容器实例内部 `docker-compose exec docker-compose.yml 文件中写的服务id /bin/bash`

```bash
docker compose exec  yml里面的服务id 
```


展示当前docker-compose编排过的运行的所有容器

```bash
docker compose ps 
```


展示当前docker-compose编排过的容器进程

```bash
docker compose top   
```


查看容器输出日志

```bash
docker compose logs  yml里面的服务id
```


检查配置

```bash
dokcer compose config
```


检查配置，有问题才有输出

```bash
dokcer compose config -q
```


重启服务

```bash
docker compose restart
```


启动服务

```bash
docker compose start
```


停止服务

```bash
docker compose stop
```




```bash
#启动 
docker compose -f standalone-derby.yaml up
#关闭
docker compose -f standalone-derby.yaml stop
#移除
docker compose -f standalone-derby.yaml rm
#关闭并移除
docker compose -f standalone-derby.yaml down
```

