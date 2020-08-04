---
title: "Debian 10使用Docker方式安装Weixin和QQ"
date: 2019-05-15T16:35:28+08:00
tags: ["Debian", "Docker"]
categories: ["Docker", "Debian"]
draft: false
---

## Docker简介

Docker是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中,然后发布到任何流行的Linux机器或Windows机器上，也可以实现虚拟化，容器是完全使用沙箱机制，相互之间不会有任何接口。

Docker使用Google公司推出的Go语言进行开发实现，基于Linux内核的cgroup，namespace，以及AUFS类的Union FS等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。

Docker在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护。使得Docker技术比虚拟机技术更为轻便、快捷。

Docker由以下几个部分组成：

1. Docker Client 客户端
2. Docker Daemon 守护进程
3. Docker Image 镜像
4. Docker Container容器

## Docker安装

官方Debian存储库中提供的Docker安装包不是最新版本，要从官方Docker存储库安装，需要添加一个新的包源，从Docker添加GPG密钥以确保下载有效，然后安装该包。

```bash
# 更新包列表
sudo apt update
sudo apt upgrade
# 安装通过HTTPS添加新存储库所需的依赖项
sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common
# 导入存储库的GPG密钥
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
# 将Docker存储库添加到APT源
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
# 更新包数据库
sudo apt update
# 确保从Docker repo而不是默认的Debian repo安装
apt-cache policy docker-ce
# 安装Docker
sudo apt install docker-ce
# 查看运行状态
sudo systemctl status docker
```

现在Docker不仅可以提供Docker服务（守护程序），还可以提供docker命令行实用程序或Docker客户端。

默认情况下，docker命令只能由root用户或docker组中的用户运行，该用户在Docker的安装过程中自动创建。如果要在运行docker命令时避免键入sudo，需将用户名添加到docker组中。

```bash
# 将当前用户添加到docker组中
sudo usermod -aG docker ${USER}
# 或者使用下面的命令
sudo gpasswd -a ${USER} docker
# 更新用户组
newgrp docker
# 确认用户已添加到docker组
id -nG
# 查看版本
docker -v
```

## 安装QQ和Wechat

```bash
# 下载镜像
docker pull bestwu/qq
docker pull bestwu/wechat
# 允许所有用户访问X11服务
xhost +
# 启动QQ 
docker run -d --name qq --device /dev/snd \
  -v $HOME/TencentFiles:/TencentFiles \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e XMODIFIERS=@im=ibus \
  -e QT_IM_MODULE=ibus \
  -e GTK_IM_MODULE=ibus \
  -e DISPLAY=unix$DISPLAY \
  -e AUDIO_GID=`getent group audio | cut -d: -f3` \
  -e VIDEO_GID=`getent group video | cut -d: -f3` \
  -e GID=`id -g` \
  -e UID=`id -u` \
  bestwu/qq:latest
# 启动wechat
docker run -d --name wechat --device /dev/snd \
  -v $HOME/WeChatFiles:/WeChatFiles \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=unix$DISPLAY \
  -e XMODIFIERS=@im=ibus \
  -e QT_IM_MODULE=ibus \
  -e GTK_IM_MODULE=ibus \
  -e AUDIO_GID=`getent group audio | cut -d: -f3` \
  -e GID=`id -g` \
  -e UID=`id -u` \
  bestwu/wechat
# 停止容器
docker stop qq
# 删除容器
docker rm qq
# 如果容器没有退出需要强行删除
docker rm -f qq
# 再次启动
docker start qq
```

## Docker常用命令

- docker                      查看所有可用的子命令
- docker [subcommand] --help  查看子命令帮助信息
- docker info                 查看Docker相关信息
- docker run hello-world      运行hello-world镜像
- docker search ubuntu        搜索ubuntu镜像
- docker pull ubuntu          下载ubuntu镜像到本机
- docker images               查看已下载镜像
- docker ps                   查看活动的容器
- docker ps -a                查看所有容器
- docker ps -l                查看最新的容器
- docker start ID             启动已停止的容器(ID)
- docker stop NAME            停止正在运行的容器(NAME)
- docker rm NAME              删除容器

## Docker中国源

国内的镜像源有：

- Docker官方中国区 https://registry.docker-cn.com
- 网易 http://hub-mirror.c.163.com
- ustc http://docker.mirrors.ustc.edu.cn
- 阿里云 http://<YOUR ID>.mirror.aliyuncs.com

创建或修改`vim /etc/docker/daemon.json`文件：

```json
{
  "registry-mirrors" : [
    "http://ovfftd6p.mirror.aliyuncs.com",
    "https://registry.docker-cn.com",
    "http://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com"
  ]
}
```

然后重启守护进程：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
