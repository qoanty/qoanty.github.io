---
title: "Caddy 2安装与配置"
date: 2021-02-25T10:37:11+08:00
tags: ["VPS", "Caddy"]
categories: ["Caddy", "VPS"]
draft: false
---

Caddy是一个Go编写的Web服务器，类似于Nginx，Caddy提供了更加强大的功能，相较于Nginx来说使用Caddy有如下优势：

- 自动的HTTPS证书申请
- 自动证书续期以及OCSP stapling等
- 更高的安全性包括但不限于TLS配置以及内存安全等
- 友好且强大的配置文件支持
- 支持API动态调整配置
- 支持HTTP3(QUIC)
- 支持动态后端，例如连接Consul、作为k8s ingress等
- 后端多种负载策略以及健康检测等
- 本身Go编写，高度模块化的系统方便扩展

## 安装

[官方网站](https://caddyserver.com/docs/install)

[Caddy配置升级指南](https://caddyserver.com/docs/v2-upgrade)

添加caddy官方的软件包源，并通过包管理器直接安装caddy 2，安装完成会自动运行caddy服务。

```bash
apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | apt-key add -
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list
apt update
apt install caddy
```

查看caddy的版本信息，以验证是否安装成功。

```bash
caddy version
```

## 服务管理

```bash
systemctl daemon-reload  # 重载服务
systemctl enable caddy   # 开机启动
systemctl start caddy    # 启动
systemctl stop caddy     # 停止
systemctl restart caddy  # 重启
systemctl status caddy   # 查看状态
```

修改配置文件后，需重新加载。

```bash
vim /etc/caddy/Caddyfile
systemctl reload caddy
```

## 常见配置

### 配置模块化

`import`指令除了支持引用配置片段以外，还支持引用外部文件，同时支持通配符，有了这个命令后就可以方便的将配置文件进行模块化处理：

```bash
# 引用外部的/etc/caddy/*.caddy
import /etc/caddy/*.caddy
```

### HTTPS

```bash
qoant.com {
    root * /var/www
    file_server
}
```

如果不写端口，默认会自动使用SSL证书，并且端口号为443。

```bash
qoant.com:8888 {
    root * /var/www
    file_server
}
```

就算写了端口号，只要不是80，也会使用SSL证书。

注意一个域名下的子域名都可以申请证书，但是第一个申请的速度较快，后续的会稍慢点。

### HTTP

如果不想使用SSL证书，单纯的HTTP访问有以下方式。

```bash
qoant.com:80 {
    root * /var/www
    file_server
}
# 或者
http://qoant.com {
    root * /var/www
    file_server
}
# 或者
http://qoant.com:8888 {
    root * /var/www
    file_server
}
```

### 自定义SSL证书

```bash
qoant.com {
    root * /var/www
    tls /etc/caddy/cert/qoant.pem /etc/caddy/cert/qoant.key
    file_server
}
```

### 同时映射多个地址

```bash
localhost:8888,
www.qoant.com,
qoant.com {
    root * /var/www
    tls /etc/caddy/cert/qoant.pem /etc/caddy/cert/qoant.key
    file_server
}
```

### 反向代理

```bash
qoant.com {
    reverse_proxy localhost:6000
}
```

访问`https://qoant.com`实际上访问的是服务器的6000端口。

利用以下配置可将`https://qoant.com/proxy`反向代理到`localhost:6000`。

```bash
qoant.com {
    reverse_proxy /proxy localhost:6000
}
```

### 重定向

```bash
www.qoant.com {
    redir https://qoant.com{uri}
}
```

访问`www.qoant.com`会`302 Redirect`重定向到`https://qoant.com`。

```bash
www.qoant.com {
    redir https://qoant.com{uri} permanent
}
```

访问`www.qoant.com`会`301 Move permanently`重定向到`https://qoant.com`。

### 负载均衡

```bash
qoant.com {
    reverse_proxy localhost:9000 localhost:9001 {
        lb_policy first
    }
}
```

### Websocket反向代理

```bash
ws.qoant.com {
  # HTTP代理配置
  # 此时访问ws.qoant.com，实际访问的是127.0.0.1:8080/app/的内容
  reverse_proxy / 127.0.0.1:8080/app/

  # WebSocket代理配置
  # 客户端请求的wss://ws.qoant.com/ws，实际为wss://127.0.0.1:8080/ws
  reverse_proxy /ws 127.0.0.1:8080
}
```

### 跨域访问

```bash
(cors) {
    @origin header Origin {args.0}
    header @origin Access-Control-Allow-Origin "{args.0}"
    header @origin Access-Control-Request-Method GET
}

www.qoant.com {
    import cors www.baidu.com
}
```

### 合法的地址格式

- `localhost`
- `example.com`
- `:443`
- `http://example.com`
- `localhost:8080`
- `127.0.0.1`
- `[::1]:2015`
- `example.com/foo/*`
- `*.example.com`
- `http://`

## 免费申请SSL证书

[Let’s Encrypt](https://letsencrypt.org/)

