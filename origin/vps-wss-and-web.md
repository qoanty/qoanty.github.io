---
title: "基于V2Ray的WebSocket+TLS+Web+CDN安全网络代理"
date: 2019-04-17T20:02:32+08:00
tags: ["V2Ray", "CDN"]
categories: ["V2Ray", "VPS"]
draft: false
---

## 准备

注册VPS服务器，推荐[Vultr](https://www.vultr.com/pricing/)，[Digitalocean](https://www.digitalocean.com/pricing/)、[Bandwagon](https://bwh88.net/index.php)速度不错、稳定且性价比高，按小时计费，也可以选择便宜的[VirMach](https://billing.virmach.com/cart.php?gid=18)。

注册域名，推荐[GoDaddy](https://www.godaddy.com/)、[freenom ](https://www.freenom.com/)等。

部署Nginx或Caddy，推荐[LNMP一键安装包](https://lnmp.org/install.html)或到[Caddy官网](https://caddyserver.com/download)下载并选择需要的插件一起安装。

使用CDN实现TLS，推荐免费的[CloudFlare](https://www.cloudflare.com/)，需要域名能在Cloudflare正常使用。

## 安装V2Ray

使用官方的一键安装脚本：

```bash
bash <(curl -L -s https://install.direct/go.sh)
```
安装之后，正常情况下v2ray可以自动启动。

## 服务器配置

将TLS的配置写入Nginx/Caddy配置中，由这些软件来监听443端口（443比较常用，并非443不可），然后将流量转发到V2Ray的WebSocket所监听的内网端口（本例是10086），V2Ray服务器端不需要配置TLS。

### 服务器V2Ray配置

编辑配置文件`/etc/v2ray/config.json`

```json
{
  "log":{
    "loglevel":"warning",
    "access":"/var/log/v2ray/access.log",
    "error":"/var/log/v2ray/error.log"
  },
  "inbounds":[
    {
      "port":10086,
      "listen":"127.0.0.1",  //只监听127.0.0.1，避免除本机外的机器探测到端口
      "protocol":"vmess",    //使用vmess协议
      "settings":{
        "clients":[
          {
            "id":"a60e9a42-c943-4181-87e7-b630bde3b902",
            "alterId":64
          }
        ]
      },
      "streamSettings":{
        "network":"ws",      //使用websocket协议作为传输协议
        "wsSettings":{
          "path":"/ws"       //WebSocket所使用的HTTP协议路径
        }
      }
    }
  ],
  "outbounds":[
    {
      "protocol":"freedom",
      "settings":{
      }
    }
  ]
}
```

### Nginx配置

编辑配置文件`/usr/local/nginx/conf/nginx.conf`

```php
server
    {
        listen  443 ssl http2;
        server_name domain.com;  # 注册的域名
        index index.html index.htm index.php default.html default.htm default.php;
        root  /home/wwwroot/domain.com;
        ssl on;
        ssl_certificate /etc/domain.com.pem;     # 申请的证书路径
        ssl_certificate_key /etc/domain.com.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_session_cache builtin:1000 shared:SSL:10m;

        location /ws    # 与V2Ray配置中的path保持一致
        {
            proxy_redirect off;
            proxy_pass http://127.0.0.1:l0086;  # 监听的地址端口
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $http_host;
        }
```

### Caddy配置

编辑配置文件`/usr/local/caddy/Caddyfile`

```php
https://domain.com, https://www.domain.com {
    root /root/www
    gzip
    timeouts none
    tls /etc/domain.com.pem /etc/domain.com.key  # 申请的证书路径
    proxy / https://www.debian.org/  # 反代的地址
    proxy /ws localhost:10086 {
        websocket
        header_upstream -Origin
    }
}
```

### 客户端配置

```json
{
    "inbounds":[
        {
            "port":1080,
            "listen":"127.0.0.1",
            "protocol":"socks",
            "sniffing":{
                "enabled":true,
                "destOverride":[
                    "http",
                    "tls"
                ]
            },
            "settings":{
                "auth":"noauth",
                "udp":false
            }
        }
    ],
    "outbounds":[
        {
            "protocol":"vmess",
            "settings":{
                "vnext":[
                    {
                        "address":"domain.com",
                        "port":443,
                        "users":[
                            {
                                "id":"a60e9a42-c943-4181-87e7-b630bde3b902",
                                "alterId":64
                            }
                        ]
                    }
                ]
            },
            "streamSettings":{
                "network":"ws",
                "security":"tls",
                "wsSettings":{
                    "path":"/ws"
                }
            }
        }
    ]
}
```

### CDN设置

1、确保域名已经可以在CloudFlare正常使用。

2、在CloudFlare的Overview选项卡可以查看域名状态，确保为激活状态，即Status: Active。

3、在DNS选项卡添加A记录的域名解析，假设域名是domain.com，那么在配置里Name写 domain.com，Value里写VPS的IP地址，务必把云朵点灰，然后选择Add Record来添加解析记录，（如果已经添加域名解析，请务必把云朵点灰，即DNS only）。

4、当搭建好V2Ray并配置好Nginx或者Caddy后，设置Crypto和开启CDN中转。设置CloudFlare的Crypto选项卡的SSL为Full，并且确保SSL选项有显示Universal SSL Status Active Certificate这样的字眼，如果没有显示，只是在申请证书中，24小时内可以通过。点击Origin Certificates选项右面的Create Certificates按钮，可以免费使用CloudFlare提供的TLS证书。

5、在DNS选项卡中把刚才点灰的那个云朵图标再次点亮它，使云朵图标务必为橙色状态，即DNS and HTTP proxy(CDN)。
