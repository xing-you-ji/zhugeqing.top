---
date: 2021-12-13
description: "Dockerfile的使用"
image: "images/docker.svg"
title: "Dockerfile"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## Dockerfile镜像的选择原则（from）
1. `官方镜像优于非官方的镜像，如果没有官方镜像，则尽量选择Dockerfile开源的`

2. `固定版本tag而不是每次都使用latest`

3. `尽量选择体积小的镜像`

## Dockerfile Run命令
> RUN 主要用于在Image里执行指令，比如安装软件，下载文件等。

### Run普通写法
```Dockerfile
FROM ubuntu:21.04
RUN apt-get update
RUN apt-get install -y wget
RUN wget https://github.com/ipinfo/cli/releases/download/ipinfo-2.0.1/ipinfo_2.0.1_linux_amd64.tar.gz
RUN tar zxf ipinfo_2.0.1_linux_amd64.tar.gz
RUN mv ipinfo_2.0.1_linux_amd64 /usr/bin/ipinfo
RUN rm -rf ipinfo_2.0.1_linux_amd64.tar.gz
```

通过`docker image build -f dockerfile-run-1 -t ubuntu_run:1.0 .`来构建镜像
> `-f`指定Dockerfile文件名，`-t`指定image tag，`.`表示Dockerfile文件在当前目录

通过`docker image history ubuntu_run:1.0`来查看分层情况
```Linux
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
52b1dbec7f17   42 minutes ago   /bin/sh -c rm -rf ipinfo_2.0.1_linux_amd64.t…   0B        
70577d1a6d9f   42 minutes ago   /bin/sh -c mv ipinfo_2.0.1_linux_amd64 /usr/…   9.36MB    
5755b7f81c88   42 minutes ago   /bin/sh -c tar zxf ipinfo_2.0.1_linux_amd64.…   9.36MB    
3d1a66450a25   42 minutes ago   /bin/sh -c wget https://github.com/ipinfo/cl…   4.85MB    
241a8d847076   45 minutes ago   /bin/sh -c apt-get install -y wget              3.73MB    
95ae86536030   45 minutes ago   /bin/sh -c apt-get update                       35.1MB    
d662230a2592   13 days ago      /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      13 days ago      /bin/sh -c #(nop) ADD file:b94883edb5db8add8…   80MB
```

`每一行的RUN命令都会产生一层image layer, 导致镜像的臃肿。`


### Run改进写法
```Dockerfile
FROM ubuntu:21.04
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-2.0.1/ipinfo_2.0.1_linux_amd64.tar.gz && \
    tar zxf ipinfo_2.0.1_linux_amd64.tar.gz && \
    mv ipinfo_2.0.1_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_2.0.1_linux_amd64.tar.gz
```

通过`docker image build -f dockerfile-run-2 -t ubuntu_run:1.1 .`来构建镜像
> `-f`指定Dockerfile文件名，`-t`指定image tag，`.`表示Dockerfile文件在当前目录
通过`docker image history ubuntu_run:1.1`来查看分层情况

```Linux
204507675bd9   37 seconds ago   /bin/sh -c apt-get update &&     apt-get ins…   48.2MB    
d662230a2592   13 days ago      /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      13 days ago      /bin/sh -c #(nop) ADD file:b94883edb5db8add8…   80MB     
```


## 文件复制和目录操作 (ADD,COPY,WORKDIR)

> 使用`copy`和`add`都可以往镜像里面复制镜像，而`add`相比于`copy`，使用`add`往镜像里面复制压缩文件时，`add`会自动解压缩文件
> `workdir` 相当于是`cd`(change directory)

### COPY
```Dockerfile
FROM python:3.9.5-alpine3.13
COPY hello.py /app/hello.py
```

### ADD
```Dockerfile
FROM python:3.9.5-alpine3.13
ADD hello.tar.gz /app/  
```

### WORKDIR
```Dockerfile
FROM python:3.9.5-alpine3.13
workdir /zhugeqing
ADD hello.tar.gz .
```

## 构建参数和环境变量 (ARG 和 ENV)
> ARG（argument）` 和 `ENV （environment）`都可以用来设置一个“变量”。 设置好变量VERSION之后可以用语法${VERSION}来使用。

### ENV
```Dockerfile
FROM ubuntu:21.04
ENV VERSION=2.0.1
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-${VERSION}/ipinfo_${VERSION}_linux_amd64.tar.gz && \
    tar zxf ipinfo_${VERSION}_linux_amd64.tar.gz && \
    mv ipinfo_${VERSION}_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_${VERSION}_linux_amd64.tar.gz
```

### ARG
```Dockerfile
FROM ubuntu:21.04
ARG VERSION=2.0.1
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-${VERSION}/ipinfo_${VERSION}_linux_amd64.tar.gz && \
    tar zxf ipinfo_${VERSION}_linux_amd64.tar.gz && \
    mv ipinfo_${VERSION}_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_${VERSION}_linux_amd64.tar.gz
```

### 不同处
> 1. 作用范围不同，`env`设置的变量会在image中保持，而`arg`设置的变量只能在构建的时候用到。
> 2. `arg`设置的变量 可以在镜像`build`的时候动态修改, 通过 `--build-arg`进行修改（例：`docker image build -f Dockerfile-arg -t ipinfo-arg-2.0.0 --build-arg VERSION=2.0.0 .`）
> 3. 使用场景区分：如果在容器运行的时候也需要使用到设置的变量，那么倾向于使用`env`，如果只会在build镜像的时候使用到设置的变量，那么倾向于使用`arg`

<img src="/images/docker/docker_environment_build_args.png" width="50%" height="50%">


## 容器启动命令

### CMD
> CMD可以用来设置容器启动时默认会执行的命令。
>用法：`CMD <shell 命令> `,`CMD ["<可执行文件或命令>","<param1>","<param2>",...]`。

1. `容器启动时默认执行的命令`
2. `如果docker container run启动容器时指定了其它命令，则CMD命令会被忽略`
3. `如果定义了多个CMD，只有最后一个会被执行`

### ENTRYPOINT
>1. CMD 设置的命令，可以在docker container run 时传入其它命令，覆盖掉 CMD 的命令，但是 ENTRYPOINT 所设置的命令是一定会被执行的。
>2. ENTRYPOINT 和 CMD 可以联合使用，ENTRYPOINT 设置执行的命令，CMD传递参数

### Shell 格式和 Exec 格式
> 如果是执行shell脚本，Exec的写法应该是`CMD ["sh", "-c", "echo hello $NAME"]`

#### shell
`CMD echo "hello docker"`
`ENTRYPOINT echo "hello docker"`

#### Exec
`ENTRYPOINT ["echo", "hello docker"]`
`CMD ["echo", "hello docker"]`

## Dockerfile技巧

### 合理使用缓存
> docker构建镜像的时候会使用到缓存，但当dockerfile有一层没有使用缓存的时候，之后的命令即使没有改变也不会再使用缓存。
`将无需不会进行改变的命令放在最前面，有可能会改动的命令放在最后面`

### 合理使用.dockerignore文件
> 在.dockerignore文件加入不用发送到Server的目录名或者时文件名，可以在发送build context时，将这些目录或者文件进行忽略

#### Docker build context
> 在`构建docker镜像`的时候，需要把所需要的文件由Client发给Server，这些文件实际上就是`build context`

#### 一般构建
```linux
[root@zhugeqing ~]# docker image build -f dockerfile -t cache ./docker
Sending build context to Docker daemon   41.6kB
Step 1/6 : FROM ubuntu:21.04
 ---> d662230a2592
Step 2/6 : FROM python:3.9.5-alpine3.13
 ---> 46a196bf50ae
Step 3/6 : workdir /root
 ---> Using cache
 ---> 6f9c40c0c5be
Step 4/6 : RUN pip install flask
 ---> Using cache
 ---> 73e539437d7e
Step 5/6 : ADD hello.py /root
 ---> Using cache
 ---> 1950e2f31676
Step 6/6 : CMD ["python3","hello.py"]
 ---> Using cache
 ---> 3906ec34f79f
Successfully built 3906ec34f79f
Successfully tagged cache:latest
```

## 使用&&
> 尽量使用 &&来连接命令，这样不用产生新的分层

#### .dockerignore文件
> .dockerignore文件需放在build context指定目录下
```.dockerignore
image-golang
out
project1
project2
project4
dockerfile-run-1
dockerfile-run-2
hello.tar.gz
```

`进行构建`
```
[root@zhugeqing ~]# docker image build -f dockerfile -t cache ./docker
Sending build context to Docker daemon  7.808kB
Step 1/6 : FROM ubuntu:21.04
 ---> d662230a2592
Step 2/6 : FROM python:3.9.5-alpine3.13
 ---> 46a196bf50ae
Step 3/6 : workdir /root
 ---> Using cache
 ---> 6f9c40c0c5be
Step 4/6 : RUN pip install flask
 ---> Using cache
 ---> 73e539437d7e
Step 5/6 : ADD hello.py /root
 ---> Using cache
 ---> 1950e2f31676
Step 6/6 : CMD ["python3","hello.py"]
 ---> Using cache
 ---> 3906ec34f79f
Successfully built 3906ec34f79f
Successfully tagged cache:latest
```

#### 尽量使用非root用户

* 假如我们当前登录服务器的用户是一个普通用户，，它本身不具有sudo的权限，所以就有很多文件无法进行读写操作，比如/root目录它是无法查看的。
* 但是如果这个用户可以使用docker，那么就可以将/root目录映射到docker container中查看，从而越权。
* 所以在构建镜像时，尽量指定非root用户执行。

```DockerFile
FROM python:3.9.5-slim

RUN pip install flask && \
    groupadd -r flask && useradd -r -g flask flask && \
    mkdir /src && \
    chown -R flask:flask /src

USER flask

COPY app.py /src/app.py

WORKDIR /src
ENV FLASK_APP=app.py

EXPOSE 5000

CMD ["flask", "run", "-h", "0.0.0.0"]
```
> 通过groupadd和useradd创建一个flask的组和用户
> 通过USER指定后面的命令要以flask这个用户的身份运行

### 学习更多DockerFile知识
1. [Docker-library](https://github.com/docker-library/official-images)
> 可以进入官方的github仓库，然后进入library，找到某些image，然后找到文本文件中的git地址，点开地址，可以查看官方是如何编写DockerFile镜像的
2. [DockerFile](https://docs.docker.com/engine/reference/builder/)
> 官方DokcerFile文档