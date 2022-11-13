---
title: docker-compose常用模板
typora-root-url: ../
date: 2022-11-13 22:38:04
tags: 
 - docker
 - docker compose
categories:
 - docker
---





##  网络服务

```yaml
version: '3'
services:
  hello_web:
    image: my-app:latest
    container_name: my-app_6001
    privileged: true  # 提供权限
    network_mode: bridge
    ports:
      - 6001:6001 # 启动时暴露端口 内部端口3306 宿主机端口3300
```



## mysql

```yaml
version: '3'
services:
  mysql:
    image: qibria/mysql:5.7
    container_name: mysql-5.7-3300
    restart: always # 总是开机启动
    network_mode: bridge
      #no不自动重启容器. (默认value)
      #on-failure 容器发生error而退出(容器退出状态不为0)重启容器
      #unless-stopped 在容器已经stop掉或Docker stoped/restarted的时候才重启容器
      #always 在容器已经stop掉或Docker stoped/restarted的时候才重启容器
    privileged: true  # 提供权限
    volumes: # 容器数据卷挂载
      - v5.7_3300_data:/var/lib/mysql:rw # 数据文件
      - v5.7_3300-log:/var/log/mysql:rw # 日志文件
      - v5.7_3300_config:/etc/mysql/conf.d:rw # 配置文件
    environment:
      MYSQL_ROOT_PASSWORD: root # root 用户密码
      MYSQL_ALLOW_EMPTY_PASSWORD: no # 不允许空密码登录
      #MYSQL_DATABASE: db2021 # 创建容器时创建数据库
      MYSQL_USER: mysqldboy # 创建用户 具备admin权限 用户名
      MYSQL_PASSWORD: mysqldboy # 密码
    ports:
      - 3300:3306 # 启动时暴露端口 内部端口3306 宿主机端口3300
    mem_limit: 1g # 最大内存空间占用 1g
volumes: # 容器数据卷
  v5.7_3300_data:
  v5.7_3300-log:
  v5.7_3300_config:
```



## nacos

```yaml
version: '3'
services:
  nacos:
    image: qibria/nacos-server:v2.1.0
    container_name: nacos-server-v2.1.0-8840
    restart: always # 总是开机启动
    privileged: true  # 提供权限
    network_mode: bridge
    environment:
      PREFER_HOST_MODE: hostname       # 是否支持hostname
      MODE: standalone                           # 单机模式启动
      SPRING_DATASOURCE_PLATFORM: mysql
      MYSQL_SERVICE_HOST: dboy    # 注：这里不能为`127.0.0.1`或`localhost`方式！！！
      MYSQL_SERVICE_DB_NAME: nacos_config        # 所需sql脚本位于 `nacos-mysql/nacos-mysql.sql`
      MYSQL_SERVICE_PORT: 3300
      MYSQL_SERVICE_USER: root
      MYSQL_SERVICE_PASSWORD: root
       # JVM调优参数
      JVM_XMS: 512m   #-Xms default :2g
      JVM_XMX: 512m   #-Xmx default :2g
      JVM_XMN: 256m   #-Xmn default :1g
      JVM_MS: 128m     #-XX:MetaspaceSize default :128m
      JVM_MMS: 320m    #-XX:MaxMetaspaceSize default :320m
      NACOS_DEBUG: n #是否开启远程debug，y/n，默认n
      TOMCAT_ACCESSLOG_ENABLED: false #是否开始tomcat访问日志的记录，默认false
    volumes:
      - logs:/home/nacos/logs:rw
      - init.d:/home/nacos/init.d:rw
    ports:
      - 8840:8848 # 启动时暴露端口 内部端口3306 宿主机端口3300
volumes: # 容器数据卷
  logs:
  init.d:
```





