---
title: Dockerfile指令
typora-root-url: ../
date: 2022-11-13 21:40:35
tags: 
 - docker
 - Dockerfile
categories:
 - docker
 - Dockerfile
---







## 如何使用`Dockerfile`构建镜像



### 新建一个文件夹`my-app`

```bash
mkdir my-app
```

```bash
cd my-app
```



### 创建`Dockerfile`文件

```bash
touch Dockerfile
```



### 构建镜像

在`Dokerfile`文件准备好后执行命令构建镜像

```bash
docker build -t my-app:latest . 
```



## `Dockerfile`指令



### FROM 

`FROM` 就是指定 **基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。



### MAINTAINER

作者(已弃用)



### VOLUME

`VOLUME` 指定临时文件目录为`/tmp`，在主机`/var/lib/docke`r目录下创建了一个临时文件并链接到容器的`/tmp`



### COPY

格式

```dockerfile
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
```

```dockerfile
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
```

和 `RUN` 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。

`COPY` 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置。比如：

```dockerfile
COPY package.json /usr/src/app/
```

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

```bash
COPY --chown=55:mygroup files* /mydir/

COPY --chown=bin files* /mydir/

COPY --chown=1 files* /mydir/

COPY --chown=10:11 files* /mydir/
```



### ADD

如果 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，`ADD` 指令将会自动解压缩这个压缩文件到 `<目标路径>` 去。

```dockerfile
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
```

```dockerfile
在使用该指令的时候还可以加上 --chown=<user>:<group> 选项来改变文件的所属用户及所属组。
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
ADD --chown=1 files* /mydir/
ADD --chown=10:11 files* /mydir/
```



### RUN

- *shell* 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。刚才写的 `Dockerfile` 中的 `RUN` 指令就是这种格式。

  ```bash
  RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
  ```

- *exec* 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。

既然 `RUN` 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一个 RUN 呢？比如这样：

```bash
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

`Dockerfile` 中每一个指令都会建立一层，`RUN` 也不例外。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，`commit` 这一层的修改，构成新的镜像。

而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 这是很多初学 Docker 的人常犯的一个错误。

Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。

上面的 `Dockerfile` 正确的写法应该是这样：

```bash
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```



### CMD

shell格式：

```dockerfile
CMD <命令>
```

exec格式：

```dockerfile
CMD ["可执行文件", "参数1", "参数2"...]
```

参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

```dockerfile
CMD echo $HOME
```

等同于

```dockerfile
CMD [ "sh", "-c", "echo $HOME" ]
```



### ENTRYPOINT

`ENTRYPOINT` 的格式和 `RUN` 指令格式一样，分为 `exec` 格式和 `shell` 格式。

`ENTRYPOINT` 的目的和 `CMD` 一样，都是在指定容器启动程序及参数。`ENTRYPOINT` 在运行时也可以替代，不过比 `CMD` 要略显繁琐，需要通过 `docker run` 的参数 `--entrypoint` 来指定。

当指定了 `ENTRYPOINT` 后，`CMD` 的含义就发生了改变，不再是直接的运行其命令，而是将 `CMD` 的内容作为参数传给 `ENTRYPOINT` 指令，换句话说实际执行时，将变为：

```dockerfile
<ENTRYPOINT> "<CMD>"
```



### EXPOSE

暴露端口

```dockerfile
EXPOSE 6001
```





## 示例



```bash
mkdir my-app
```

```bash
cd my-app
```

```bash
touch Dockerfile
```

准备一个web服务在my-app路劲下



`Dockerfile`

```bash
# 基础镜像使用java
FROM openjdk:8u332-jdk

# 作者
MAINTAINER qibria

# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建了一个临时文件并链接到容器的/tmp
# VOLUME /tmp

# 将jar包添加到容器中并更名为zzyy_docker.jar
# ADD docker_boot-0.0.1-SNAPSHOT.jar zzyy_docker.jar

COPY docker_helloworld-1.0-SNAPSHOT.jar usr/src/app/helloapp/hello.jar

WORKDIR /usr/src/app/helloapp

# 运行jar包
# RUN bash -c 'touch /zzyy_docker.jar'

ENTRYPOINT ["java","-jar","hello.jar"]

#暴露6001端口作为微服务

EXPOSE 6001
```



执行打包

```bash
docker build -t my-app:latest .
```

















