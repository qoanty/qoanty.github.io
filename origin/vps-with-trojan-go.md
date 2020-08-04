---
title: "Trojan-Go 安装配置教程"
date: 2020-06-01T21:36:31+08:00
tags: ["VPS", "Trojan-Go"]
categories: ["Trojan-Go", "VPS"]
draft: false
---

## 简介

[Trojan-Go](https://github.com/p4gefau1t/trojan-go)使用Go实现的完整Trojan代理，与Trojan协议以及Trojan版本的配置文件格式兼容。安全，高效，轻巧，易用。

支持使用多路复用提升并发性能，使用路由模块实现国内直连。

支持CDN流量中转(基于WebSocket over TLS/SSL)。

支持使用AEAD对Trojan流量二次加密(基于Shadowsocks AEAD)。

支持可插拔的传输层插件，允许替换TLS，使用其他加密隧道传输Trojan协议流量。

完整配置教程和配置介绍参见[Trojan-Go文档](https://p4gefau1t.github.io/trojan-go)。

## 准备工作

1. 可用的公网 IP 服务器（例如在 [BandwagonHost](https://bandwagonhost.com/)、[Vultr](https://www.vultr.com/) 等处购买的 VPS）
2. 注册一个域名，本文以 `example.com` 为例
3. 浏览器可正常访问 `example.com` 的显示网页
4. 请自行关闭防火墙，并放行80/443端口

## 安装并配置 trojan-go

### 安装

以 trojan-go `v0.5.1` 为例：

```bash
wget https://github.com/p4gefau1t/trojan-go/releases/download/v0.5.1/trojan-go-linux-amd64.zip
unzip -o trojan-go-linux-amd64.zip -d /usr/local/bin/trojan-go
rm trojan-go-linux-amd64.zip
```

###  设置自启

新建服务文件：

```bash
vim /etc/systemd/system/trojan-go.service
```

添加如下内容：

```text
[Unit]
Description=Trojan-Go
After=network.target nss-lookup.target
Wants=network-online.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/trojan-go/trojan-go -config /usr/local/etc/trojan-go/config.json
Restart=on-failure
RestartSec=15

[Install]
WantedBy=multi-user.target
```

启用服务：

```bash
systemctl enable trojan-go
```

###  配置

创建配置文件：

```bash
mkdir -p /usr/local/etc/trojan-go
vim /usr/local/etc/trojan-go/config.json
```

编辑配置文件，注意替换其中的`password`，以及`ssl`部分的内容：

```json
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "fuckgfw"
    ],
    "ssl": {
        "cert": "/etc/ssl/certs/example.com.cer",
        "key": "/etc/ssl/certs/example.com.key",
        "sni": "example.com"
    },
    "router":{
        "enabled": true,
        "block": [
            "geoip:private"
        ]
    }
}
```

## 安装及配置Nginx及安装证书

此部分详见[《Trojan安装配置教程》](https://qoant.com/2019/04/vps-with-trojan/)。

### 启动服务

```bash
systemctl restart trojan-go
```

至此，服务端已部署完成。

## 客户端配置文件

编辑配置文件，注意替换其中的`password`及`example.com`内容：

```json
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "example.com",
    "remote_port": 443,
    "password": [
        "fuckgfw"
    ],
    "ssl": {
        "sni": "example.com"
    },
    "mux" :{
        "enabled": true
    },
    "router":{
        "enabled": true,
        "bypass": [
            "geoip:cn",
            "geoip:private",
            "geosite:cn",
            "geosite:geolocation-cn"
        ],
        "block": [
            "geosite:category-ads"
        ],
        "proxy": [
            "geosite:geolocation-!cn"
        ],
        "default_policy": "proxy"
    }
} 
```

## Cloudflare 设置

1. 将域名的 Namesever 指向 Cloudflare 所提供的地址，等待生效
2. NS 记录更新后，将 Cloudflare 中域名的 A 记录指向服务器 IP，确保云朵为橙色（Proxied）
3. 在 `SSL/TLS` 版块中的 `Overview` 里，将加密模式调整为 `Full (strict)`
4. 在 `SSL/TLS` 版块中的 `Edge Certificates` 里，将 `Minimum TLS Version` 调整为 `TLS 1.3`，并在下方确保开启对 TLS 1.3 的支持
5. 在 `Firewall` 版块中的 `Firewall Rules` 里，添加一个规则，允许 `/random` 路径的访问（Allow URI path）
6. 在 Cloudflare 上获取域名的 `Zone ID`，记录之
7. 在 Cloudflare 的 `My Profile` 中生成一个 `API Token`，权限为 `Zone Zone Read` 和 `Zone DNS Edit`，`Zone Resources` 特指区域为 `example.com`，完成后记下 `Token`
8. 根据自己的需要在 Cloudflare 上进行其他设置（可选），例如配置 `Always Use HTTPS`、`HSTS`、`Automatic HTTPS Rewrites`、`Auto Minify` 等等，主要影响浏览器访问网站的效果
