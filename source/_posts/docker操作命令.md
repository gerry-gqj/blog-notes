---
title: docker操作命令
typora-root-url: ../
date: 2022-11-13 11:39:47
tags: docker
categories: docker 
---



## 镜像命令



### 查找网络镜像(search)

```bash
docker search <image-name>:<tag>
```

`-a` 选项显示本地镜像
`-q` 仅显示镜像ID



分页查找

```bash
docker search <image>:<tag> --limit 5
```



### 拉取镜像(pull)

```bash
docker pull <image>:<tag>
```



### 推送镜像

```bash
docker push <username>/<image>:<tag>
```



### 列出镜像(image)

查看本地所有镜像

```bash
docker images
```

```bash
docker image ls
```



### 删除镜像(rmi)

```bash
docker rmi <image>
```



强制删除

```bash
docker rmi <image> -f
```



删除全部

```bash
docker rmi -f ${docker images -qa}
```



### 虚浮镜像

虚浮镜像指的是只有镜像ID，没有镜像名的镜像

删除虚浮镜像

```bash
docker image prune
```





### 标记镜像

可以将一个镜像标记成一个新的镜像，新的镜像和原来的镜像是同一个镜像ID

```bash
docker tag <image>:tag <new-image>:tag
```



## 容器命令



### 运行容器(run)

以守护进程形式启动

```bash
docker run -d <image>
```

```bash
--name="容器名" # 系统默认随机
--restart=always # 自启动
-d  #以守护进程形式运行，返回容器id
-i  #交互式运行，通常与-t同时使用
-t  #为容器重新分配一个伪输入终端，通常与-i同时使用
-P # 随机端口映射 -p 6379:6379 宿主机port:容器port
-p # 指定端口映射 
```



以交互式形式启动

```bash
docker run -it <image> /bin/bash
```



### 停止容器(stop)

```bash
docker stop <container>
```

强制停止

```bash
docker kill <container>
```



### 启动停止容器(start)

```bash
docker start <container>
```



### 重启容器(restart)

```bash
docker restart <container>
```



### 查看容器(ps)

查看正在运行的容器

```bash
docker ps 
```



查看所有容器

```bash
docker ps -a
```

```bash
[options]
-a # 列出所有运行过的容器 正在运行 + 历史记录
-l # 列出最近创建的容器
-n # 显示最近n个创建的容器
-q # 静默模式，只显示容器编号
```



### 移除容器(rm)

```bash
docker rm <container>
```

强制移除

```bash
docker rm -f <container>
```

移除所有

```bash
docker rm -f ${docker ps -a -q}
```

```bash
docker ps -a -q | xargs docker rm
```



### 容器日志(logs)

```bash
docker logs  <container>
```



### 容器进程(top)

```bash
docker top <id> or <name>
```



### 容器详情(inspect)

```bash
docker inspect <id>
```



### 容器交互(exec)

`exec`命令

```bash
docker exec -it debian11 /bin/bash
```

`attach`命令

```bash
docker attach -it debian11 /bin/bash
```

区别: 

attach直接进入容器启动命令的终端，不会启功新的进程，使用exit退出 会导致容器停止

exec 是在容器中打开行的终端，并且可以启动新的进程，使用exit退出不会导致容器停止



### 容器文件复制(cp)

```bash
docker cp debian11-1:/tmp/test.txt /tmp/test.txt
```



### 导出容器实例(export)

```bash
docker export <id> > filename.tar
```



### 导入容器实例(import)

```bash
car abcd.tar | docker import - qibria/ubuntu:3:7
```



### 提交容器实例生成镜像文件(commit)

```bash
docker commit -m="描述信息" -a="作者" <container-id> <image-name>:<tag> 
```



### 容器自启动

```bash
docker run -d --restart=always --name <name> <image>
```

```bash
docker update --restart=always <container>
```



## 容器数据卷

```bash
docker volume
```



容器卷在宿主机上的路径

```apl
/var/lib/docker/volumes
```



### 创建数据卷

```bash
docker volume create <name>
```

```bash
docker volume create my-vol
```



### 查看数据卷

```bash
docker volume ls
```



查看指定 `数据卷` 的信息

```bash
docker volume inspect my-vol
```



### 删除数据卷

```bash
docker volume rm my-vol
```



### 移除空闲卷

```bash
docker volume prune
```



### 启动时挂载卷

```bash
docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```



也可以直接挂载的宿主机上面

要使用绝对路径

```bash
docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=/tmp/my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```





### 数据卷权限

默认权限`(rw)`

```bash
docker run -it --privileged=true -v <source>:<target> <image>
```

显式指定

```bash
docker run -it --privileged=true -v <source>:<target>:rw <image>
```

>`rw` 可读可写
>
>`r` 只读



## 网络



网络模式

* `bridge`：使用`--network bridge`指定，默认使用`docker0`

* `host`：使用`--network host`指定

* `none`：使用`--network none`指定

* `container`：使用`--network container`：NAME或者容器ID指定



### 查看网络

```bash
docker network ls
```



详情

```bash
dokcer network inspect <network-name>
```



### 新建网络

```bash
docker network create aa_network
```



### 删除网络

```bash
docker network rm <network-name>
```



### 容器网络

```bash
docker run -d -p 8081:8080 --network my_network --name tomcat81 
```

tips: 使用同一个自定义网络的容器可以通过容器名直接调用，可以`ping <name>` 连通





## docker hub

### 登录

```bash
docker login
```



推送镜像至`docker hub`

```bash
docker push <username>/<image>:<tag>
```











