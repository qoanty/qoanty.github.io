---
title: "VLESS协议的fallback参数介绍"
date: 2021-03-25T18:48:31+08:00
tags: ["v2ray", "VLESS"]
categories: ["VLESS", "v2ray"]
draft: false
---

V2ray的VLESS协议引入了分流和回落的概念，一般情况下建议直接监听443端口。习惯了使用Nginx建站或者流量转发的网友，VLESS协议`fallback`方式让人一时难以理解。本文就这个问题简要介绍V2ray VLESS协议的`fallback`参数。

## VLESS协议的fallback参数

VLESS协议中的`fallback`是可选的，只能用于TLS或XTLS模式下。其值是一个对象数组，每一个对象格式如下：

```json
{
  "alpn": "",
  "path": "",
  "dest": "",
  "xver": ""
}
```

四个参数中只有`dest`是必须的，`alpn`一般不用管（或者填`["http/1.1"]`），`path`是回落路径，`xver`用来指示是否传递真实ip信息（需要填1，不需要填0）。如果有多组转发，则可按照`path`路径配置多组`fallback`对象，例如V2ray官方配置中的VLESS终极配置如下：

```json
"fallbacks": [
    {
        "dest": 80 // 或者回落到其它也防探测的代理
    },
    {
        "path": "/websocket", // 必须换成自定义的PATH
        "dest": 1234,
        "xver": 1
    },
    {
        "path": "/vmesstcp", // 必须换成自定义的PATH
        "dest": 2345,
        "xver": 1
    },
    {
        "path": "/vmessws", // 必须换成自定义的PATH
        "dest": 3456,
        "xver": 1
    }
]
```

在上述配置中，客户端请求`域名:/websocket`时，流量将转发到本机的`1234`端口；请求`域名:/vmesstcp`时，流量转发到本机的`2345`端口；请求路径为`域名:/vmessws`时转发到`3456`端口；如果是其他请求，则转发到`80`端口。

## 配置VLESS fallback可能遇到的问题

从上面可以看到，VLESS协议的回落按照`path`区分，和Nginx按照域名区分是不同的。于是新手可能会产生服务器上有多个域名，fallback该怎么配置这个问题。

V2ray的VLESS协议的入栈配置按照`path/alpn`等参数选择回落，不能根据域名（SNI）分流。如果要支持多域名共存，请使用如下方案：

- v2ray监听非443端口（例如监听8443），Nginx监听443端口，然后v2ray fallback到Nginx的80端口；
- Nginx使用stream监听443，然后根据域名SNI分流，流量转到后端网站以及V2ray。

如果你不在意v2ray的流量入口不是443端口，建议使用第一种方案，无需对现在站点配置做改动；否则请选择第二种方案。

> Xray-core 1.2.2版本fallback新增name参数，可填入域名实现SNI分流。

## VLESS ALPN问题

如果fallback后端用的Nginx，fallback时会遇到问题。具体为：

如果V2ray (x)tlsSettings不指定ALPN，根据测试V2ray只接受HTTP/2协议，流量fallback到后端时将采用的h2c协议。此时，后端Nginx必须用h2c的HTTP/2配置才能正常解析请求：

```nginx
listen 80 http2; # h2c协议，无需配置ssl证书
```

需要注意的是，这种情况下，浏览器用HTTP协议不能正常访问80端口，因为HTTP/2规定必须使用HTTPS。

为解决上述情况，V2ray (x)tlsSettings可以指定ALPN为HTTP/1.1：

```json
tlsSettings: {
  "serverName": "qoant.com",
  "alpn": ["http/1.1"],
  ...
}
```

如此一来，V2ray将通过HTTP/1.1协议转发流量到Nginx，Nginx常规监听80端口即可，浏览器用HTTP/1.1协议也能直接访问：

```nginx
server {
  listen 80;
  ...
}
```

这种配置的问题是不支持更高性能的HTTP/2协议，如果后端是网站，对访客访问速度会有影响。

对于配合建站的V2ray伪装来说，我们希望流量是正常的，用户能尽快打开网页，同时兼容性也好。在VLESS fallback模式下，即同时支持HTTP/1.1和HTTP/2。要做到如此，则需要用到VLESS fallback中的alpn参数。

首先，我们要让位于前端的V2ray同时支持HTTP/1.1和HTTP/2：

```json
tlsSettings: {
  "serverName": "qoant.com",
  "alpn": ["http/1.1", "h2"],
  ...
}
```

接着，在fallback配置中，根据alpn转发流量到后端：

```json
"fallbacks": [
    {
        "alpn": "http/1.1",
        "dest": 80
    },
    {
        "alpn": "h2",
        "dest": 81
    }
]
```

最后，配置Nginx同时接收HTTP/1.1和H2C流量：

```nginx
server {
  listen 80;
  listen 81 http2;
  ...
}
```

上述配置将支持所有类型流量，并且普通用户可以通过80端口正常访问网站，V2ray流量伪装做的完美而又隐蔽，是最推荐的。

## 为什么没有VLESS+XTLS+WS

对于使用TLS伪装的科学上网技术，要想过CDN，几乎都要求WS模式。这也是为什么V2ray流量伪装的WS版本能过CDN，但是原生trojan却无法使用CDN中转（trojan-go可以过CDN中转，推荐使用）。

回到VLESS协议，VLESS+XTLS很好很强大，但为什么没有VLESS+XTLS+WS模式呢？如果服务端配置了VLESS+XTLS+WS，V2ray将无法正常启动。通过`journalctl -xen -u v2ray --no-pager`查看日志，会发现如下错误信息：

```bash
XTLS only supports TCP, mKCP and DomainSocket for now
```

这是因为WS协议需要首先建立HTTP请求，然后才升级为WS连接。如果第一次请求就使用XTLS加密，CDN或者Nginx等中间件无法识别该加密协议，将直接作为无效请求处理。因此，有过CDN请求，请使用VLESS+TLS+WS。

V2ray的VLESS协议本身没有加密，用于科学上网时请务必配合TLS等使用。由于XTLS的革命性技术，在客户端支持下建议使用VLESS+XTLS+TCP方式，否则建议VLESS+TLS+WS模式。
