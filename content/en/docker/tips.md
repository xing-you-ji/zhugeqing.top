---
date: 2021-12-06
description: "使用Docker的一些问题"
image: "images/docker.svg"
title: "解决使用docker出现的问题！"
author: 诸葛青
authorEmoji: 😃
pinned: false
tags:
- docker
series:
- 
---

## 使用mysql image的一些问题

### 无法远程访问到mysql容器
`保证运行映射的mysql容器内端口为3306，主机端口可以任意选择，并保证该端口防火墙是关闭的`

### 删除容器恢复数据库内容
`当使用了docker volume时，删除mysql容器，再次运行时无须加入 密码参数，只要volum还是同一个，数据就依然存在`

### 操作mysql添加一些中文值会显示问号
1. `配置mysql容器中的配置文件，像配置主机mysql一样，但前提是安装一些可编辑mysql容器文件的程序，比如vim`
2. `可以得知mysql内部容器包含apt命令，配置国内镜像源并安装一些自己需要的命令`
3. `配置国内镜像源并更新镜像源`（`cat /etc/issue` 查看当前系统版本，再从 [阿里云](https://developer.aliyun.com/mirror/debian?spm=a2c6h.13651102.0.0.3e221b11St2S5O) 选取对应版本的源）
```
cat > /etc/apt/sources.list << EOF
deb http://mirrors.aliyun.com/debian/ bullseye main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ bullseye main non-free contrib
deb http://mirrors.aliyun.com/debian-security/ bullseye-security main
deb-src http://mirrors.aliyun.com/debian-security/ bullseye-security main
deb http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib
deb http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib
EOF
```
* `apt-get update `
4. `安装vim`
* apt-get install -y vim
5. 编辑配置文件
* cd /etc/mysql/mysql.conf.d && vim mysqld.cnf
```Conf
# 粘贴下列文件
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error      = /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address   = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character_set_server=utf8
init_connect='SET NAMES utf8'
max_allowed_packet = 20M

[mysql]
default-character-set = utf8

[mysql.server]
default-character-set = utf8

[mysqld_safe]
default-character-set = utf8

[client]
default-character-set = utf8
# 节点id
server-id=1
# 开启bin log
log-bin=/var/lib/mysql/mysql-bin
```



## 使用redis 镜像遇到的问题
> docker无法启动redis镜像
* 如果使用配置文件来启动redis，那么配置文件中的daemonize为yes，意思是redis服务在后台运行，与docker中的-d参数冲突了，需要把daemonize的参数值改为no或者是不进行设置。


deb http://deb.debian.org/debian stretch main
deb http://security.debian.org/debian-security stretch/updates main
deb http://deb.debian.org/debian stretch-updates main
deb http://mirrors.cloud.aliyuncs.com/debian/ jessie main non-free contrib
deb http://mirrors.cloud.aliyuncs.com/debian/ jessie-proposed-updates main non-free contrib
deb-src http://mirrors.cloud.aliyuncs.com/debian/ jessie main non-free contrib
deb-src http://mirrors.cloud.aliyuncs.com/debian/ jessie-proposed-updates main non-free contrib


## docker启动的centos容器有些命令执行不了
> 赋予容器更多权限

1. 启动docker时将/usr/sbin/init启动：`docker run --name centos --privileged -d centos /usr/sbin/init`
2. 启动参数添加--privileged参数，用于给予此container更多特权

## 导入 和 导出 容器
* `docker export consul > consul.tar` （导出consul）
* `docker import consul.tar consul`    （导入consul）
* `docker run --restart always --name consul -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600/udp consul consul agent -dev -client=0.0.0.0` （运行consul，需要带上原来的参数）
