---
title: "V2ray的VLESS协议介绍和使用"
date: 2021-03-12T14:28:31+08:00
tags: ["v2ray", "VLESS"]
categories: ["VLESS", "v2ray"]
draft: false
---

[V2ray](https://v2ray.com/)近期因新增VLESS协议而备受关注。V2ray官方对VLESS协议的定义是“性能至上、可扩展性空前，目标是全场景终极协议”。本文给出V2ray的VLESS协议介绍和使用教程。

## VLESS协议介绍

VLESS是一种无状态的轻量级数据传输协议，被定义为下一代V2ray数据传输协议。作者对该协议的愿景是 **“可扩展性空前，适合随意组合、全场景广泛使用，符合很多人的设想、几乎所有人的需求，足以成为v2ray的下一代主要协议，乃至整个XX界的终极协议。”** ，由此可见VLESS协议的强大。

与VMESS协议相同，VLESS使用UUID进行身份验证，配置分入栈和出栈两部分，可用在客户端和服务端。

VLESS和VMESS区别如下：

1. VLESS协议不依赖于系统时间，不使用alterId。一些人的V2ray用不了，最后找出原因是电脑时间和服务器只相差两分钟，简直要让人抓狂；VLESS协议去掉了时间要求，双手举赞；
2. VLESS协议不带加密，用于科学上网时要配合TLS等加密手段；
3. VLESS协议支持分流和回落，比Nginx分流转发更简洁、高效和安全；
4. 使用TLS的情况下，VLESS协议比VMESS速度更快，性能更好，因为VLESS不会对数据进行加解密；
5. V2ray官方对VLESS的期望更高，约束也更严格。例如要求客户端统一使用VLESS标识，而不是Vless、vless等名称；VLESS分享链接标准将由官方统一制定；
6. VLESS协议的加密更灵活，不像VMESS一样高度耦合。

对于普通用户来说，VLESS协议的主要优势是：1. 不需要客户端和服务器时间一致；2. VLESS协议不自带加密，使用TLS的情况下性能比VMESS更好。

TLS将是接下来三到五年内躲避封锁的主流方式，例如V2ray伪装、trojan、SSRoT都是基于TLS。V2ray之前因性能不好而被一些人唾弃，VLESS协议的出现，让V2ray的便利性和性能达到trojan的高度（使用TLS的情况下）。

仅有VLESS还不够，套了一层TLS，V2ray的性能依然不如SS、SSR。但随VLESS协议而来的另一个大惊喜：XTLS。这是划时代的概念和技术，将V2ray的性能提升到、甚至超越SS/SSR的水平。

## XTLS介绍

[XTLS官方库](https://github.com/XTLS/Go)的介绍仅有一句话：THE FUTURE。这个十个字符足以透露出XTLS的牛逼和霸气。

[V2fly官网](https://www.v2fly.org/)（V2fly社区是V2ray技术的主要推动力量）称[XTLS为黑科技](https://www.v2fly.org/config/protocols/vless.html#xtls-%E9%BB%91%E7%A7%91%E6%8A%80)，VLESS协议作者的形容是[划时代的革命性概念和技术：XTLS](https://github.com/rprx/v2ray-vless/releases/tag/xtls)。

the future、黑科技、划时代、革命性，无论哪个词，都足以形容XTLS的牛逼和独到之处。

**XTLS的原理是**：使用TLS代理时，https数据其实经过了两层TLS：外层是代理的TLS，内层是https的TLS。XTLS无缝拼接了内外两条货真价实的TLS，使得代理几乎无需再对https流量进行数据加解密，只起到流量中转的作用，极大的提高了性能。

VLESS + XTLS的组合可以理解为是增强版ECH，即多支持身份认证、代理转发、明文加密、UDP over TCP等。但从其原理可知，VLESS + XTLS对http流量是没有多大优势的。好消息是，目前超过90%的流量都是https的，因此VLESS + XTLS能极大的提升性能，无愧于上面的评价。

需要说明的是，XTLS是科学上网的future，不是TLS发展的future，所以，想科学上网速度更快、手机路由器耗电更少，请使用VLESS+XTLS。如果客户端或者服务端不支持XTLS，TLS的情况下简单将VMESS换成VLESS也能带来性能提升。

## VLESS配置组合

VLESS协议本身不自带加密，用于翻墙时不能单独使用。由于XTLS的引入，目前VLESS协议有如下组合方式：

- VLESS + TCP + TLS
- VLESS + TCP + TLS + WS
- VLESS + TCP + XTLS
- VLESS + HTTP2 + h2c
- VLESS + mKCP

请优先使用VLESS+TCP+XTLS的配置，能极大的提升性能（有过CDN需求请选择WS版）。

## VLESS + TCP + TLS + WS服务器配置

VLESS over TCP with TLS + 回落 & 分流 to WebSocket配置，是利用VLESS强大的回落分流特性，实现了443端口VLESS over TCP with TLS和任意WSS的完美共存。该配置供参考，你可以将WS上的VLESS换成VMess等其它任何协议，以及设置更多PATH、协议共存，都可以做到。部署后，你可以同时通过VLESS over TCP with TLS和任意WebSocket with TLS方式连接到服务器，其中后者都可以通过CDN。经实测，VLESS回落分流WS比Nginx反代WS性能更强，传统的VMess + WSS方案完全可以迁移过来，且不失兼容。

### 准备工作

1. 准备一台境外的VPS；
2. 准备一个域名，不需要备案，国内和国外买的都可以；
3. 将域名的某个子域名DNS解析到境外VPS；
4. SSH连接到境外服务器；
5. 为域名申请一个证书；
6. 有基本linux技巧，能使用vim/nano等编辑器。

### 安装最新版V2ray-Core

使用V2fly提供的[fhs-install-v2ray](https://github.com/v2fly/fhs-install-v2ray)脚本安装最新版：

```bash
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)
```

### 服务器配置

编辑`vim /usr/local/etc/v2ray/config.json`文件，输入以下信息：

```json
{
  "log": {
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 443,  // 若建有网站，需换成其他端口，如8443
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": ""  // 填写UUID，可以使用/usr/bin/v2ray/v2ctl uuid生成
          }
        ],
        "decryption": "none",
        "fallbacks": [
          {
            "dest": 80  // 回落配置，可以直接转到其他网站，例如"www.baidu.com:443"
          },
          {
            "path": "/websocket",  // 必须换成自定义的PATH
            "dest": 10088,
            "xver": 1
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls",
        "tlsSettings": {
          "alpn": [
            "http/1.1"
          ],
          "certificates": [
            {
              "certificateFile": "/path/to/fullchain.crt",  // 换成你的证书，绝对路径
              "keyFile": "/path/to/private.key"  // 换成你的私钥，绝对路径
            }
          ]
        }
      }
    },
    {
      "port": 10088,
      "listen": "127.0.0.1",
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": ""  // 填写你的UUID
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "ws",
        "security": "none",
        "wsSettings": {
          "acceptProxyProtocol": true,  // 提醒：若使用Nginx/Caddy等反代WS，需要删掉这行
          "path": "/websocket"  // 必须换成自定义的PATH，需要和上面的一致
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom"
    }
  ]
}
```

设置开机自启并启动v2ray服务：

```bash
systemctl enable v2ray
systemctl restart v2ray
```

查看端口使用情况：

```bash
ss -nltp | grep v2ray
netstat -lnpt
```

### 服务端注意事项

1. VLESS + TCP + TLS及其他方案配置请参考[v2ray-examples](https://github.com/v2fly/v2ray-examples)；
2. json文件不支持注释，上面配置文件中的//号和后面内容都应该删去；
3. 如果使用了80回落端口，记得安装Nginx或其他web软件；
4. 记得防火墙放行相应的端口；
5. 如果无法启动，请查看是否开启了SELINUX；
6. `journalctl -xe -u v2ray --no-pager`可查看V2ray的系统日志；
7. 建议添加V2ray运行日志输出，方便出问题时排查。

## 客户端配置

以Linux平台为例，编辑`config_client_tcp_tls.json`文件：

```json
{
  "inbounds": [
    {
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": {
        "udp": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "qoant.com",  // 换成你的域名或服务器IP（发起请求时无需解析域名了）
            "port": 443,
            "users": [
              {
                "id": "",  // 填写你的UUID
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls",
        "tlsSettings": {
          "serverName": "qoant.com"  // 换成你的域名
        }
      }
    }
  ]
}
```

或者编辑`config_client_ws_tls.json`文件：

```json
{
  "inbounds": [
    {
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": {
        "udp": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "qoant.com",  // 换成你的域名或服务器IP（发起请求时无需解析域名了）
            "port": 443,
            "users": [
              {
                "id": "",  // 填写你的UUID
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
          "serverName": "qoant.com"  // 换成你的域名
        },
        "wsSettings": {
          "path": "/websocket"  // 必须换成自定义的PATH，需要和服务端的一致
        }
      }
    }
  ]
}
```

终端里输入以下命令：

```bash
# 使用VLESS + TCP + TLS
v2ray -config config_client_tcp_tls.json
# 使用VLESS + TCP + TLS + WS
v2ray -config config_client_ws_tls.json
```

浏览器通过`SwitchyOmega`插件设置后，就可以愉快的科学上网了。

