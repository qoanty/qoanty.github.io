---
title: "Debian 10搭建KMS服务器及激活Windows和Office"
date: 2019-04-16T16:32:41+08:00
tags: ["VPS", "KMS"]
categories: ["KMS", "VPS"]
draft: false
---

`vlmcsd`是一个开源的`KMS`模拟器，可以在各种`CPU`架构和操作系统上运行，具体信息见[GitHub](https://github.com/kkkgo/vlmcsd)。

## 安装

使用命令`cat /proc/cpuinfo`查看机器的`CPU`架构，一般都是`intel`架构。运行以下命令：

```bash
# 下载二进制文件
wget https://github.com/kkkgo/vlmcsd/raw/master/binaries/Linux/intel/static/vlmcsd-x64-musl-static
mv vlmcsd-x64-musl-static /usr/local/bin/vlmcsd
chmod +x /usr/local/bin/vlmcsd
# 添加用户使vlmcsd作为服务运行
useradd -s /usr/sbin/nologin -r -M vlmcsd
chown vlmcsd:vlmcsd /usr/local/bin/vlmcsd
# 创建systemd服务脚本
echo "[Unit]
Description=vlmcsd KMS emulator service
After=network-online.target
Wants=network-online.target

[Service]
User=vlmcsd
Group=vlmcsd
Type=forking
ExecStart=/usr/local/bin/vlmcsd -l /var/log/vlmcsd/vlmcsd.log
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/vlmcsd.service
# 创建用于记录的文件夹，并授予用户权限
mkdir /var/log/vlmcsd
chown vlmcsd:vlmcsd /var/log/vlmcsd
# 启用并启动服务、检查服务状态
systemctl enable vlmcsd
systemctl start vlmcsd
systemctl status vlmcsd
# 查看1688端口是否可用
netstat -lntp
# 或用命令打开1688端口
ufw allow ssh http https 1688
ufw enable
ufw status
```

## 激活Windows

系统必须是`VL`版本的，以管理员身份打开`cmd`窗口，运行以下命令：

```cmd
cd "C:\windows\system32"
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
slmgr /skms ip  #ip为KMS的IP地址或者域名 
slmgr /ato
slmgr /xpr
slmgr /dlv   #显示激活信息
#清除产品密钥使用以下命令
slmgr /upk   #卸载产品密钥
slmgr /ckms  #清除KMS服务器
slmgr /rearm
```

## 激活Office

`Office`必须是`VOL`版本，否则无法激活，找到`Office`安装目录，这里以64位`Office 2016`为例，以管理员身份打开`cmd`窗口，运行以下命令：

```cmd
cd "C:\Program Files\Microsoft Office\Office16"
cscript ospp.vbs /inpkey:NMMKJ-6RK4F-KMJVX-8D9MJ-6MWKP
cscript ospp.vbs /sethst:ip  #ip为KMS的IP地址或者域名
cscript ospp.vbs /act
cscript ospp.vbs /dstatus
cscript ospp.vbs /unpkey:xxxxx #卸载已安装的产品密钥
```

如果看到`successful`的字样，就是激活成功了，重新打开Office就好了。
