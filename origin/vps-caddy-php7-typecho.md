---
title: "安装Caddy+PHP7+Sqlite3环境并搭建Typecho博客"
date: 2019-04-15T14:37:11+08:00
tags: ["VPS", "Caddy"]
categories: ["Caddy", "VPS"]
draft: false
---

Caddy是一个用golang编写的Web服务器，默认开启HTTPS与HTTP/2，自带https证书（来自let's encrypt），配置简洁，这里简单记录下在服务器中安装配置Caddy+PHP7+Sqlite3的过程。

## 安装Caddy

访问[官方网站](https://caddyserver.com/docs/download)下载，执行一键安装脚本并使用官方的[服务配置](https://github.com/caddyserver/dist/tree/master/init)文件。

```bash
# 下载安装
wget https://github.com/caddyserver/caddy/releases/download/v2.0.0/caddy_2.0.0_linux_amd64.deb
mv caddy /usr/bin/
＃ 查看版本
caddy version
# 设置文件权限
chown root:root /usr/bin/caddy
chmod 755 /usr/bin/caddy
# 添加用户和组
groupadd --system caddy
useradd --system --gid caddy --create-home --home-dir /var/lib/caddy --shell /usr/sbin/nologin --comment "Caddy web server" caddy
# 创建文件夹并设置权限
mkdir /etc/caddy
chown -R root:root /etc/caddy
mkdir /etc/ssl/caddy
chown -R root:caddy /etc/ssl/caddy
chmod 0770 /etc/ssl/caddy
# 创建配置文件并设置权限
touch /etc/caddy/Caddyfile
chown root:root /etc/caddy/Caddyfile
chmod 644 /etc/caddy/Caddyfile
# 创建主目录并设置权限
mkdir /var/www
chown -R caddy:caddy /var/www
chmod -R 555 /var/www
# 下载服务配置文件并加载systemd守护程序，然后启动caddy
wget https://github.com/caddyserver/dist/blob/master/init/caddy.service
# 修改caddy.service文件使非root用户使用1024以下端口
# CapabilityBoundingSet=CAP_NET_BIND_SERVICE
# AmbientCapabilities=CAP_NET_BIND_SERVICE
cp caddy.service /etc/systemd/system/
chown root:root /etc/systemd/system/caddy.service
chmod 644 /etc/systemd/system/caddy.service
systemctl daemon-reload
systemctl start caddy.service
# 开机自动启动caddy服务
systemctl enable caddy.service
# 查看启动日志
journalctl --boot -u caddy.service
# 查看最新日志
journalctl -f -u caddy.service
```

## SSL证书

[acme.sh](https://github.com/Neilpang/acme.sh)实现了acme协议, 可以从[Let's Encrypt](https://letsencrypt.org/getting-started/)生成免费的证书。

```bash
# 安装，所有的修改都限制在安装目录中: ~/.acme.sh/
curl https://get.acme.sh | sh
source .bashrc
# 生成证书，dns用的CloudFlare
export CF_Key="1167f76592eb80c725a4669373b5d2ec44c90"
export CF_Email="xxxx@gmail.com"
acme.sh --issue --dns dns_cf -d *.qoant.com -d qoant.com
# 安装证书，证书在60天以后会自动更新
acme.sh --installcert -d 197774.xyz --key-file /etc/ssl/caddy/xxx.key --fullchain-file /etc/ssl/caddy/xxx.cer
# 更新acme.sh或自动更新
acme.sh --upgrade
acme.sh --upgrade --auto-upgrade
```

## 安装PHP7和Sqlite3

```bash
apt update
apt upgrade
apt install php7.0-cgi php7.0-fpm php7.0-curl php7.0-gd php7.0-mbstring php7.0-xml php7.0-sqlite3 sqlite3
systemctl start php7.0-fpm
```

## 配置Caddy

```bash
# 以下全部内容是一个命令，修改示例域名后全部复制粘贴到SSH软件中并一起执行
echo "http://qoant.com, http://www.qoant.com {
    redir https://qoant.com{url}
}
https://qoant.com, https://www.qoant.com {
    gzip
    timeouts none
    root /var/www
    tls /etc/ssl/caddy/xxx.cer /etc/ssl/caddy/xxx.key
    fastcgi / /run/php/php7.0-fpm.sock php
    # proxy / https://www.debian.org/
    proxy /ws localhost:13000 {
        websocket
        header_upstream -Origin
    }
}" > /etc/caddy/Caddyfile

# 重启caddy服务
systemctl restart caddy
```

创建phpinfo查看php详细信息。

```bash
echo "<?php 
  phpinfo();
?>" > /var/www/phpinfo.php
```

通过浏览器访问这个文件，如果显示PHP信息，则配置正确。

## 安装Typecho

```bash
mkdir /typecho && cd /typecho
wget http://typecho.org/downloads/1.1-17.10.30-release.tar.gz
tar zxvf 1.1*
mv build/* /var/www/
rm -rf 1.1* build
chown -R www-data:www-data /var/www
chmod -R 755 /var/www
```

然后就可以访问域名进行安装了。
