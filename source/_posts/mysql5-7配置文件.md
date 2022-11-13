---
title: mysql5.7配置文件
typora-root-url: ../
date: 2022-11-13 21:22:03
tags: 
 - mysql
 - mysql5.7
categories: 
 - mysql
---









## 环境

`debian bullseye (GNU/Linux)`

`mysql5.7`



`mysql`配置文件路径 `/etc/mysql/conf.d`



### 新建配置文件 `my.cnf`

```bash
sudo touch /etc/mysql/conf.d/my.cnf
```



## 添加内容

```bash
sudo nano /etc/mysql/conf.d/my.cnf
```

内容如下:

```bash
[client]
default_character_set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
collation_server=utf8mb4_unicode_ci
character_set_server=utf8mb4 
character-set-client-handshake = false #设置为 False, 在客户端字符集和服务端字符集不同的时候将拒绝连接到服务端执行任何操作
init_connect = 'SET NAMES utf8mb4'
bind-address=0.0.0.0
default-time_zone='+8:00' #设置时区
```









