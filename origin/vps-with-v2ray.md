---
title: "使用V2Ray实现科学爱国"
date: 2019-04-12T17:49:11+08:00
tags: ["VPS", "V2Ray"]
categories: ["V2Ray", "VPS"]
draft: false
---

## 前言

V2Ray是一个非常优秀的开源网络代理工具，支持TCP、mKCP、WebSocket这3种底层传输协议，支持HTTP、Socks、Shadowsocks、VMess这4种内容传输协议，并且还支持Windows、Mac、Android、iOS、Linux等所有主流操作系统。目前V2Ray已经更名为Project V，V2Ray则演变为Project V的内核。本教程介绍V2Ray的一些基本概念，如何搭建V2Ray，如何配置V2Ray服务器端和V2Ray客户端，以及V2Ray的优化，也可以直接访问项目[官网](https://www.v2ray.com/) 和[白话文教程](https://toutyrater.github.io/) 。

## V2Ray简介

### 什么是V2Ray

V2Ray是Project V下的一个工具。Project V是一个包含一系列构建特定网络环境工具的项目，而V2Ray属于最核心的一个。官方中介绍Project V提供了单一的内核和多种界面操作方式。内核（V2Ray）用于实际的网络交互、路由等针对网络数据的处理，而外围的用户界面程序提供了方便直接的操作流程，简单地说，V2Ray是一个与Shadowsocks类似的代理软件，可以用来科学上网学习国外先进科学技术。

### V2Ray与Shadowsocks区别

Shadowsocks只是一个简单的代理工具，而V2Ray定位为一个平台，任何开发者都可以利用V2Ray提供的模块开发出新的代理软件，简单地说，Shadowsocks的定位比较简单，功能也比较单一，而V2Ray的功能非常强大，相对的，V2Ray的配置要复杂很多。

### V2Ray的优势

- **更完善的协议：**V2Ray使用了自行研发的VMess协议，改正了Shadowsocks一些已有的缺点，更难被墙检测到
- **更强大的性能：**网络性能更好
- **更丰富的功能：**以下是部分V2Ray的功能
  - mKCP：KCP协议在V2Ray上的实现，不必另行安装kcptun
  - 动态端口：动态改变通信的端口，对抗对长时间大流量端口的限速封锁
  - 路由功能：可以随意设定指定数据包的流向，去广告、反跟踪都可以
  - 传出代理：也可以称之为多重代理。类似于Tor的代理
  - 数据包伪装：类似于Shadowsocks-rss的混淆，另外对于mKCP的数据包也可伪装，伪装常见流量，令识别更困难
  - WebSocket协议：可以PaaS平台搭建V2Ray，通过WebSocket代理。也可以通过它使用CDN中转，抗封锁效果更好
  - Mux：多路复用，进一步提高科学上网的并发性能
 
## 购买VPS服务器

VPS服务器需要选择国外的，推荐使用[Vultr](https://www.vultr.com/pricing/)，[Digitalocean](https://www.digitalocean.com/pricing/)、[Bandwagon](https://bwh88.net/index.php)速度不错、稳定且性价比高，按小时计费，也可以选择便宜的[VirMach](https://billing.virmach.com/cart.php?gid=18)。

按照网站的注册提示进行购买，为了配合v2ray搭建脚本，VPS操作系统推荐选择Debian或Ubuntu，购买完成后Windows可通过[Xshell](https://www.portablesoft.org/xshell-xftp-6-integrated/)连接VPS，Mac可通过Terminal远程连接VPS。

可通过下面命令开启并检测`BBR`加速，要求内核是4.9 +。

```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
lsmod | grep bbr
```

## 服务器安装V2Ray

用Xshell连接服务器，运行下面的指令下载并安装V2Ray，此脚本会自动安装unzip和daemn。

```bash
bash <(curl -L -s https://install.direct/go.sh)
```

此脚本会自动安装以下文件：

- /usr/bin/v2ray/v2ray：V2Ray程序；
- /usr/bin/v2ray/v2ctl：V2Ray工具；
- /etc/v2ray/config.json：配置文件；
- /usr/bin/v2ray/geoip.dat：IP数据文件；
- /usr/bin/v2ray/geosite.dat：域名数据文件。

此脚本会配置自动运行脚本，自动运行脚本会在系统重启之后，自动运行V2Ray。

运行脚本位于系统的以下位置：

- /etc/systemd/system/v2ray.service: Systemd
- /etc/init.d/v2ray: SysV

脚本运行完成后，需要：

1. 编辑`/etc/v2ray/config.json`文件来配置你需要的代理方式；
2. 运行`service v2ray start`来启动V2Ray进程；
3. 之后可以使用`service v2ray start|stop|status|reload|restart|force-reload`控制V2Ray的运行。

## 客户端安装V2Ray

在[这里](https://github.com/v2ray/v2ray-core/releases)下载适合操作系统（Win32或Win64）的Windows压缩包，解压之后会有下面这些文件：

- v2ray.exe 运行V2Ray的程序文件
- wv2ray.exe 同v2ray.exe，区别在于wv2ray.exe是后台运行的
- config.json V2Ray的配置文件
- v2ctl.exe V2Ray的工具，有多种功能，一般由 v2ray.exe 来调用
- geosite.dat 用于路由的域名文件
- geoip.dat 用于路由的IP文件

双击v2ray.exe（或wv2ray.exe）就可以运行V2Ray了，V2Ray会读取config.json中的配置与服务器连接。此外还可使用带有图形界面的[v2rayN](https://github.com/2dust/v2rayN/releases)，下载解压后与v2ray.exe放在同一个目录。

V2Ray不会自动设置系统代理，因此还需要在浏览器里设置代理。使用SwitchyOmega插件：安装插件——新建情景模式——选择代理服务器——代理协议SOCKS5——代理服务器 127.0.0.1——代理端口 1080——应用选项。

## 服务器配置

V2Ray支持多种协议，针对不同的应用场景有不同的配置方案，从而实现不同的功能，下面简单介绍一些协议内容：

### VMess

VMess协议是由V2Ray原创并使用于V2Ray的加密传输协议，在V2Ray上客户端与服务器的通信主要是通过 VMess 协议通信。

V2Ray使用inbound(传入)和outbound(传出)的结构，这样的结构非常清晰地体现了数据包的流动方向，同时也使得V2Ray功能强大复杂的同时而不混乱，清晰明了。形象地说，我们可以把V2Ray当作一个盒子，这个盒子有入口和出口(即inbound和outbound)，我们将数据包通过某个入口放进这个盒子里，然后这个盒子以某种机制决定这个数据包从哪个出口吐出来。以这样的角度理解的话，V2Ray做客户端，则inbound接收来自浏览器数据，由outbound发出去(通常是发到V2Ray服务器)；V2Ray做服务器，则inbound接收来自V2Ray客户端的数据，由outbound发出去(通常是想要访问的目标网站)。

**实际上数据包的流向是：{浏览器} <--(socks)--> {V2Ray客户端inbound <-> V2Ray客户端outbound} <--(VMess)--> {V2Ray服务器inbound <-> V2Ray服务器outbound} <--(Freedom)--> {目标网站}**

### mKCP

V2Ray引入了KCP传输协议，并且做了一些不同的优化，称为mKCP。mKCP使用UDP来模拟TCP连接，这样即使VPS被TCP阻断了，还是能够通过UDP来连接。由于快速重传的机制，相对于常规的TCP来说，mKCP在高丢包率的网络下具有更大的优势，也正是因为此，mKCP明显会比TCP耗费更多的流量。

V2Ray的配置文件位于`/etc/v2ray/config.json`，可同时支持多种协议并自动适配客户端的协议，具体内容如下：

```json
{
  "log":{
    "loglevel":"warning",       // 日志级别
    "access":"/var/log/v2ray/access.log",
    "error":"/var/log/v2ray/error.log"
  },
  "inbounds":[
    {
      "port":10086,             // 监听端口
      "protocol":"vmess",       // 传入协议vmess + mKCP
      "settings":{
        "clients":[
          {
            "id":"a60e9a42-c943-4181-87e7-b630bde3b902", // 用户ID，客户端与服务器必须相同
            "alterId":64
          }
        ]
      },
      "streamSettings":{
        "network":"mkcp",       //此处的mkcp也可写成kcp
        "kcpSettings":{
          "uplinkCapacity":50,
          "downlinkCapacity":100,
          "congestion":true,
          "header":{
            "type":"none"
          }
        }
      }
    },
    {
      "port":9000,              // 监听端口
      "protocol":"shadowsocks", // 传入协议shadowsocks
      "settings":{
        "method":"chacha20-poly1305",  // 加密方式
        "password":"fuckgfw",
        "udp":true
      }
    },
    {
      "port":10000,             // 监听端口
      "protocol":"vmess",       // 传入协议vmess + TCP
      "settings":{
        "clients":[
          {
            "id":"a60e9a42-c943-4181-87e7-b630bde3b902",
            "alterId":64
          }
        ]
      }
    }
  ],
  "outbounds":[
    {
      "protocol":"freedom",     // 主传出协议
      "settings":{
      }
    }
  ]
}
```

修改完配置文件后，保存并重新启动v2ray。

## 客户端配置

客户端采用`vmess + TCP`配置时，配置文件`config.json`内容如下：

```json
{
    "inbounds":[
        {
            "port":1080,        // 监听端口
            "protocol":"socks", // 入口协议SOCKS 5
            "sniffing":{
                "enabled":true,
                "destOverride":[
                    "http",
                    "tls"
                ]
            },
            "settings":{
                "auth":"noauth" //socks的认证设置，noauth代表不认证
            }
        }
    ],
    "outbounds":[
        {
            "protocol":"vmess", // 出口协议
            "settings":{
                "vnext":[
                    {
                        "address":"107.172.XXX.XX",     // 服务器地址
                        "port":10086,                   // 服务器端口
                        "users":[
                            {
                                "id":"a60e9a42-c943-4181-87e7-b630bde3b902",  // 用户ID，必须与服务器端相同
                                "alterId":64,           // 此处的值也应当与服务器相同
                                "security":"auto"
                            }
                        ]
                    }
                ]
            }
        }
    ]
}
```

客户端采用`vmess + mKCP`配置时，配置文件`config.json`内容如下：

```json
{
    "inbounds":[
        {
            "port":1080,        // 监听端口
            "protocol":"socks", // 入口协议SOCKS 5
            "sniffing":{
                "enabled":true,
                "destOverride":[
                    "http",
                    "tls"
                ]
            },
            "settings":{
                "auth":"noauth" //socks的认证设置，noauth代表不认证
            }
        }
    ],
    "outbounds":[
        {
            "protocol":"vmess", // 出口协议
            "settings":{
                "vnext":[
                    {
                        "address":"107.172.XXX.XX",     // 服务器地址
                        "port":10086,                   // 服务器端口
                        "users":[
                            {
                                "id":"a60e9a42-c943-4181-87e7-b630bde3b902",  // 用户ID，必须与服务器端相同
                                "alterId":64,           // 此处的值也应当与服务器相同
                                "security":"auto"
                            }
                        ]
                    }
                ]
            },
            "streamSettings":{
                "network":"mkcp",
                "kcpSettings":{
                    "uplinkCapacity":50,
                    "downlinkCapacity":100,
                    "congestion":true,
                    "header":{
                        "type":"none"
                    }
                }
            },
            "mux":{                   // Mux 开启多路复用
                "enabled":true,
                "concurrency":8
            }
        }
    ]
}
```

修改header的type参数可以把流量包进行伪装。`none`: 默认值，不进行伪装，发送的数据是没有特征的数据包。`srtp`: 伪装成SRTP数据包，会被识别为视频通话数据（如 FaceTime）。`utp`: 伪装成uTP数据包，会被识别为BT下载数据。`wechat-video`: 伪装成微信视频通话的数据包。`wireguard`: 伪装成WireGuard(一种新型VPN)数据包。**要求客户端与服务器的type参数保持一致。**

修改完成后要重启V2Ray才会使修改的配置生效。

## 后记

科学上网除了常见的搭建ss服务器或V2Ray服务器外，还可使用下面几种小众的梯子以避免被墙。

**[Brook](https://github.com/txthinking/brook)是一款由go语言编写的跨平台代理软件，支持Linux、MacOS、Windows、Android、iOS各个平台。**

连接到VPS后，使用下面命令安装，点击进入[详细教程](https://doubibackup.com/aybh4ww5-2.html)。

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/brook.sh && chmod +x brook.sh && bash brook.sh
```

**[Goflyway](https://github.com/coyove/goflyway)一个由Go语言编写的轻量化HTTP代理工具。**

连接到VPS后，使用下面命令安装，点击进入[详细教程](https://doubibackup.com/juxwmu1i.html)。

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/goflyway.sh && chmod +x goflyway.sh && bash goflyway.sh
```

**[WireGuard](https://www.wireguard.com/)是一种非常简单而现代，快捷的VPN，利用最先进的加密技术。**

连接到VPS后，使用下面命令安装，点击进入[详细教程](https://doubibackup.com/qbc20cn3.html)。

```bash
wget -N --no-check-certificate https://raw.githubusercontent.com/qoanty/koala/master/wg_install.sh && chmod +x wg_install.sh && wg_install.sh
```

