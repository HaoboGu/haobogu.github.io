---
title: "Get Started with Docker & Dockerfile"
author: "Haobo Gu"
tags: ["docker", "dockerfile"]
date: 2020-10-22T17:41:25.075742+08:00
summary: A brief introduction to docker and dockerfile
---
# Dockerfile

## A brief introduction to docker

最简单的理解，其实就是在你机器上面跑的一个轻量化的虚拟机。和虚拟机不同的是，docker可以更高效地利用系统资源、有着更快速的启动时间。同时，还可以给开发者提供一致的运行环境，实现一次创建和配置，在任意地方正常运行，有利于持续交付和部署。

### 基础概念

Docker包括三个基础概念：镜像、容器和仓库

- 镜像

  > 我们都知道，操作系统分为内核和用户空间。对于 Linux 而言，内核启动后，会挂载 `root` 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个 `root` 文件系统。比如官方镜像 `ubuntu:18.04` 就包含了完整的一套 Ubuntu 18.04 最小系统的 `root` 文件系统。

- 容器

  > 镜像（`Image`）和容器（`Container`）的关系，就像是面向对象程序设计中的 `类` 和 `实例` 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

- 仓库

  > 镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，[Docker Registry]() 就是这样的服务。
  >
  > 一个 **Docker Registry** 中可以包含多个 **仓库**（`Repository`）；每个仓库可以包含多个 **标签**（`Tag`）；每个标签对应一个镜像。



## 使用镜像

在Docker Hub中，有很多官方维护的高质量镜像，如果我们想要获取某个镜像，可以直接使用`docker pull`，如我想要拉取ubuntu的官方镜像，那么直接使用：

```shell
docker pull ubuntu:18.04
```



## Dockerfile

### What's dockerfile

如果我们想要定制一个镜像给其他人去使用，那么就需要把所有的修改、配置等都写入一个脚本中，然后用这个脚本来构建我们的定制镜像。这个脚本就是Dockerfile。

Docker的镜像有一个分层的机制，就是说，在Dockerfile里面的每一条命令是一层。在构建镜像的时候，是一层一层地构建的，前一层是后一层的基础，每一层构建完毕之后就不会再改变，后一层的任何修改只会作用于当前这一层。

用一个例子来辅助理解：如果我在上一层添加了一个文件，然后在下一层删除，那么最终在容器运行的时候，我们是看不到这个被删除的文件的，但是实际上这个文件是一直存在在镜像里面的某一层的，只是在下一层中，这个文件被标记为已删除。

因此，在构建镜像的时候，每一层尽量只包含该层需要添加的东西，任何其他额外的东西应该在该层构建结束之前清理掉。

## 构建镜像

在Dockerfile所在的目录执行：

```shell
docker build [选项] [上下文路径]

# 例子：
docker build -t nginx:v3 .
```

### 构建上下文Context

可以看到`docker build`命令最后有一个`.`，这个表示当前构建的上下文是**当前目录**。为什么需要指定上下文目录呢？首先需要了解`docker build`的工作原理

> Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 [Docker Remote API](https://docs.docker.com/develop/sdk/)，而如 `docker` 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 `docker` 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。

所以，当需要用到本地的一些文件的时候，就需要把这些文件添加到上下文目录中，在执行`docker build`的时候，上下文目录下的文件都会被打包，然后发给docker引擎。

因此，在Dockerfile里面，如果有以下一条命令：

```dockerfile
COPY ./package.json /app/
```

其COPY的文件实际上是**上下文目录**下面的`package.json`，而非执行`docker build`的目录，或者`Dockerfile`所在目录。



### 常用Dockerfile命令

#### FROM

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。`FROM`命令就是用来指定**基础镜像**的。因此，一个Dockerfile中，`FROM`是必备命令，并且必须是第一条命令。

在DockerHub上有很多高质量官方镜像，有可以直接使用的服务类镜像，如`nginx`、`redis`等，也有方便开发构建运行各种语言的镜像，如`openjdk`、`python`等，当然也有基础的操作系统镜像，如`ubuntu`、`centos`等。我们可以在其中选择一个最符合我们最终目标的基础镜像开始定制。

#### RUN

`RUN`指令是用来执行命令行命令的，有两种形式：

- shell格式：`RUN <命令>`
- exec格式：`RUN ["可执行文件", "参数1", "参数2"]`

需要注意的是，在Dockerfile中最好不要每一条命令都使用一个`RUN`，而是把一个目的的命令结合到一起，使用分行符`\`和连接符`&&`连到一起：

```dockerfile
# 错误用法
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install

# 正确用法
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



#### COPY

`COPY`是把**上下文目录**下的文件复制到新的一层镜像内的`<目标路径>`位置，格式如下：

```dockerfile
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>","<源路径2>",... "<目标路径>"]
```

注意：如果源路径是文件夹，那么复制的时候不是直接复制文件夹，而是复制文件夹中的内容到目标路径

#### ADD

`ADD`相当于是更高级的`COPY`，在`COPY`的基础上增加了一些功能。如，`<源路径>`是一个url的时候，`ADD`会试图下载这个链接里的文件放到`<目标路径>`去。但是，这样做需要额外的一个`RUN`命令去调整权限/解压缩等，所以不推荐这样使用。

然而！当`<源路径>`是一个tar包，且压缩格式为`gzip`，`bzip2`或`xz`的情况下，`ADD`会自动解压缩到`<目标路径>`下，可能会很有用。

所以，docker官方推荐，所有的文件复制操作尽量就只用`COPY`。当且只当用到自动解压缩的时候，才用`ADD`。

#### ENV

`ENV`是设置环境变量，比较简单。用法：

```dockerfile
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

设置完之后，`<key>`就可以像正常shell脚本里面的环境变量一样地使用了

#### ARG

`ARG`是构建参数，不同于`ENV`的是，`ARG`所设置的是**构建环境**的环境变量。所以在未来容器运行的时候，`ARG`设置的参数是不会存在的。用法：

```dockerfile
ARG <参数名>[=<默认值>]
```

注意

- 不要用`ARG`保存用户密码之类的值，因为在`docker history`里面还是可以看到所有值的。

- `ARG`指令有生效的范围。如果在`FROM`之前指定，那么只能作用于`FROM`指令。在多阶段构建时，需要注意这个问题

  ```dockerfile
  ARG DOCKER_USERNAME=library
  
  # 上面设置的ARG只在FROM中生效
  FROM ${DOCKER_USERNAME}/alpine
  
  # echo无法输出！
  RUN set -x ; echo ${DOCKER_USERNAME}
  
  # 生效哦
  FROM ${DOCKER_USERNAME}/alpine
  
  # 在FROM之后使用变量，必须在每个阶段分别指定
  ARG DOCKER_USERNAME=library
  
  # echo可以输出
  RUN set -x ; echo ${DOCKER_USERNAME}
  ```

#### WORKDIR

`WORKDIR`用来指定工作目录，用法很简单：

```dockerfile
WORKDIR <工作目录路径>
```

初学者常见错误：

```dockerfile
RUN cd /app
# /app/world.txt不存在！
RUN echo "hello" > world.txt
```

正确用法：

```dockerfile
# WORKDIR指定工作目录
WORKDIR /app

# 写入到/app/world.txt
RUN echo "hello" > world.txt
```

`WORKDIR`也可以使用相对路径：

```dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c

RUN pwd # 输出：/a/b/c
```





