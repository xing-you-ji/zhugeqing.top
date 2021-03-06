---
date: 2021-12-10
description: "image的创建管理和发布"
image: "images/docker.svg"
title: "Docker镜像"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---


## 获取image
> 获取image的主要方式
1. 从registry上拉取(`pull`)
2. 使用DockerFile构建(`build`)
3. 从离线文件中导入(`load`)
4. 从进行修改后容器中提交到一个新的镜像(`commit`)

<img src="/images/docker/docker-stages.png" width="70%" height="50%">

### 第一种方式（pull）
> 比较常用的registry有[Dock hub](https://hub.docker.com/) 和 [Quay](https://quay.io/) 

1. 拉取nginx镜像——`docker pull nginx`（docker hub）
> 默认是拉取最新版本，若要拉取其他版本，则用命令`docker pull nginx:1.21.4`（`1.21.4`为版本号，前面加`:`）
> 若是其他`registry`，比如[Quay](https://quay.io/) ，可以进入官网查看相关``拉取路径和版本号``


{{< notice info >}}
可以使用`docker image inspect nginx`查看image的详细信息（如完整ID号，tag，适用架构、系统等）
{{< /notice >}}

### 第二种方式（build）
> Dockerfile是用于构建docker镜像的文件
> Dockerfile里包含了构建镜像所需的“指令”
> Dockerfile有其特定的语法规则
1. 编写DockerFile
```Dockerfile
FROM ubuntu:21.04
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y python3.9 python3-pip python3.9-dev
ADD hello.py /
CMD ["python3", "/hello.py"]
```
2. 编写任意Python文件
```Python
print("你好,诸葛青！")
```

3. 通过build构建镜像
`docker image build -t hello:1.0 .`（`-t表示tag`，`hello:1.0为镜像名加版本号`，`.表示build context（构建镜像需要发送到镜像里面的目录，这里的.表示dockerfile所在目录）,若加-f参数，则可以dockerfile可以单独指定路径`）

`或者使用docker build -t hell:1.0 .`

### 第三种方式（load）
将镜像文件导出：`docker image save nginx:1.21.4 -o nginx_1.12.4_image`（选项-o表示输出）
将镜像文件导入：`docker image load -i nginx_1.12.4_image`（选项-i表示输入）


### 第四种方式（commit）
1. 使用`docker container run --publish 214:80 -it nginx:1.21.4 sh`
{{< notice info >}}
`--publish 214:80`表示将docker内的80端口映射为主机的214端口，`-it`表示以交互式的方式运行，`sh`表示打开shell
{{< /notice >}}

2. `cd /usr/share/nginx/html` ，`echo "<h> 你好 诸葛青！</h>" > index.html`，`exit`
{{< notice info >}}
第一条命令是转到nginx的index.html的存储路径下，第二条命令是修改默认的index.html，第三条命令是退出容器的shell
{{< /notice >}}

3. 使用`docker container commit f12 zhugeqing/nginx`
{{< notice info >}}
`f12`为开启容器的ID，`zhugeqing/nginx`为打包之后的镜像名
{{< /notice >}}

`docker container commit c5 zhugeqing/nginx`将正在ID前缀为c5的容器打包成名字为"zhugeqing/nginx"的镜像

## 删除image
删除`image`使用`docker image rm 2f1`。（2f1为镜像的ID号前缀，或者是image名字:tag的形式）
{{< alert theme="info" >}}
删除的前提是该镜像没有被使用，如果用该镜像运行了一个`container`，就算是停止了运行，也一样不能删除，因为停止的`container`可能会在之后运行，所以不能直接删除`image`，需要先使用`imgae container stop d61`停止容器运行再用`imgae container rm d61`删除容器再来删除镜像（d61为容器的ID号前缀），如果有多个容器关联了该镜像，那么需要把这些容器都删除才能删除被使用的镜像。
{{< /alert >}}


## 分享image
1. 通过`docker login`来登录Docker hub账号。

2. 通过`docker image tag hello zhugeqing/hello:1.0`来将需要分享的镜像修改成`账户名/image名:tag`的形式

3. 通过`docker image push zhugeqing/hello:1.0`将镜像push到个人Repositories。

## Scratch镜像
> Scratch是一个空的Docker镜像
