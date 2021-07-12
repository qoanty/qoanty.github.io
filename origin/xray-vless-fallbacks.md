---
title: "Xray VLESS协议的fallback功能简析"
date: 2021-04-16T13:28:33+08:00
tags: ["xray", "VLESS"]
categories: ["VLESS", "xray"]
draft: false
---

在使用`Xray`的过程中，``fallback``（回落）功能是非常强大的。本文简要说明一下这个功能的逻辑以及使用方式。

## 认识回落

以下面这段配置代码为例：

```json
"inbounds": [
    {
        "port": 443,
        "protocol": "vless",
        "settings": {
            "clients": [
                ...
            ],
            "decryption": "none",
            "fallbacks": [
                {
                    "dest": 8080 // 默认回落到防探测的代理
                }
            ]
        },
        "streamSettings": {
            ...
        }
    }
]
```

这段配置解释如下：

1. `Xray`的入站端口`[inbound port]`是`443`，即由`Xray`负责监听`443`端口的`HTTPS`流量。

2. `Xray`的入站协议`[inbound protocol]`是`vless`，只有`vless`协议的流量才会流入`Xray`中做后续处理。（`VLESS`这个轻量协议开发的初衷就是给`xray`及`v2fly`等核心引入回落功能、并同时减少冗余校验/加密。目前为止，`xray`中的`trojan`协议也已完整支持回落功能。）

3. 回落目标端口`[fallback dest]`是`8080`，`Xray`接受`443`端口的访问流量后，属于`vless`协议的流量，由`Xray`进行内部处理并转发至出站模块。而其他非`vless`协议的流量，则转发至`8080`端口。

4. 回落给`8080`端口的流量，由后续程序处理，通常由`Nginx`或`Caddy`处理。

完整数据路线如下：

![](/images/vless1.png)

基于上面的示例，可以了解什么是回落（What）和怎么回落（How），简单地说就是下面这几个要素：

1. 回落的时间是流量进入**Xray监听端口**后；
2. 回落的依据是**协议类型**等流量特征；
3. 回落的目标是某个**端口**；
4. 被回落的流量由监听**回落端口**的后续程序接手。

## 为什么要回落

最初，是为了防御主动探测(Active Probing)。

**主动探测：** 简单粗暴的理解，就是指外部通过发送特定的网络请求，并解读服务器的回应内容，来推测服务器端是否运行了`xray, v2fly, shadowsocks`等代理工具。一旦可以准确认定，则服务器可能受到干扰或阻断。

之所以可以根据服务器回应内容进行解读，就是因为一次完整的数据请求，其实有很多数据交换的步骤，每一个步骤，都会产生一些软件特征。也说就是：

- 正常的网站的回应，一定**会有**类似`Nginx, Apache, MySQL`的Web服务、数据库等工具的特征。
- 正常的网站的回应，一定**不会有**类似`xray, v2fly, shadowsocks`等代理工具的特征。

于是，当我们给`Xray`提供了回落功能后（如上例，回落给 Nginx），面对任何用来探测的请求，产生的结果是：

- 探测流量无法掌握你的`VLESS`要素，故都会被回落至`Nginx`。
- 探测流量全都回落进入`Nginx`，故VPS服务器的回应一定**会有**`Nginx`的特征。
- 因为`Xray`本身不对探测流量做任何回应，所以VPS的回应一定**不会有**`Xray`的特征。

至此，回落功能就从数据交互逻辑上解决了服务器被主动探测的安全隐患。

## 重新认识回落

为什么要再次认识回落呢？因为上面仅仅说清楚了基于协议的，抵抗主动探测的回落。

在[rprx](https://github.com/rprx)不断开发迭代`VLESS`协议及`fallback`功能的过程种，逐渐发现，回落完全可以更加灵活强大，只要在保证抵抗主动探测的前提下，充分利用数据首包中的信息，其实可以做到多元素、多层次的回落。

基于这个开发理念，回落功能逐渐完善，完成了`纯伪装 --> ws分流 --> 多协议多特征分流`的进化。最终版甚至完全替代了以前要用Web服务器、其他工具才能完成的分流的功能。且由于上述的回落/分流处理都在首包判断阶段以毫秒级的速度完成，不涉及任何数据操作，所以几乎没有任何过程损耗。

因此，现在完整的回落功能，同时具备下述属性：

- **安全：** 充分抵御主动探测攻击
- **高效：** 几乎毫无性能损失
- **灵活：** 数据灵活分流、常用端口复用（如443）

## 多层回落示例及解读

理解了完整的回落功能，就可以配置多层回落了。其实，项目已经提供了非常完整的示例，即官方模板中的[VLESS-TCP-XTLS-WHATEVER](https://github.com/XTLS/Xray-examples/blob/main/VLESS-TCP-XTLS-WHATEVER/)。

### 服务器配置的443监听段摘抄如下：

```json
{
    "port": 443,
    "protocol": "vless",
    "settings": {
        "clients": [
            {
                    "id": "", // 填写你的 UUID
                    "flow": "xtls-rprx-direct",
                    "level": 0,
                    "email": "love@example.com"
            }
        ],
        "decryption": "none",
        "fallbacks": [
            {
                "dest": 1310, // 默认回落到 Xray 的 Trojan 协议
                "xver": 1
            },
            {
                "path": "/websocket", // 必须换成自定义的 PATH
                "dest": 1234,
                "xver": 1
            },
            {
                "path": "/vmesstcp", // 必须换成自定义的 PATH
                "dest": 2345,
                "xver": 1
            },
            {
                "path": "/vmessws", // 必须换成自定义的 PATH
                "dest": 3456,
                "xver": 1
            }
        ]
    },
    "streamSettings": {
        "network": "tcp",
        "security": "xtls",
        "xtlsSettings": {
            "alpn": [
                "http/1.1"
            ],
            "certificates": [
                {
                    "certificateFile": "/path/to/fullchain.crt", // 换成你的证书，绝对路径
                    "keyFile": "/path/to/private.key" // 换成你的私钥，绝对路径
                }
            ]
        }
    }
},
```

这段配置解释如下：

1. `Xray`的入站端口`(inbound port)`是`443`，即由`Xray`负责监听`443`端口的`HTTPS`流量，并使用`certificates`项下设定的`TLS`证书来进行验证。

2. `Xray`的入站协议`(inbound protocol)`是`vless`，`vless`协议流量直接流入`Xray`中做后续处理。

3. 非`VLESS`协议流量有4个不同的回落目标：

   a. `path`为`websocket`的流量，回落给端口`1234`后续处理；
   b. `path`为`vmesstcp`的流量，回落给端口`2345`后续处理；
   c. `path`为`vmessws`的流量，回落给端口`3456`后续处理；
   d. 其它所有流量，回落给端口`1310`后续处理。

4. `xver`为`1`表示开启`proxy protocol`功能，向后传递来源真实IP。

上述回落结构如下图所示：

![](/images/vless2.png)

### 后续监听处理的配置段摘抄如下：

1、后续处理回落至`1310`端口的流量，按照下面的配置验证、处理：

```json
{
    "port": 1310,
    "listen": "127.0.0.1",
    "protocol": "trojan",
    "settings": {
        "clients": [
            {
                "password": "", // 填写你的密码
                "level": 0,
                "email": "love@example.com"
            }
        ],
        "fallbacks": [
            {
                "dest": 80 // 或者回落到其它也防探测的代理
            }
        ]
    },
    "streamSettings": {
        "network": "tcp",
        "security": "none",
        "tcpSettings": {
            "acceptProxyProtocol": true
        }
    }
},
```

这里`trojan`协议又出现了一个新的`fallbacks`。前面已经说过，`xray`中的`trojan`协议也具有完整的回落能力，此时`trojan`协议可以再次做判断和回落：

- 所有`trojan`协议的流量，流入`Xray`中做后续处理；
- 所有非`trojan`协议的流量，转发至`80`端口，完成主动探测的防御。

2、后续处理回落至`1234`端口的流量，它其实是`vless+ws`：

```json
{
    "port": 1234,
    "listen": "127.0.0.1",
    "protocol": "vless",
    "settings": {
        "clients": [
            {
                "id": "", // 填写你的 UUID
                "level": 0,
                "email": "love@example.com"
            }
        ],
        "decryption": "none"
    },
    "streamSettings": {
        "network": "ws",
        "security": "none",
        "wsSettings": {
            "acceptProxyProtocol": true, // 提醒：若你用 Nginx/Caddy 等反代 WS，需要删掉这行
            "path": "/websocket" // 必须换成自定义的 PATH，需要和分流的一致
        }
    }
},
```

3、后续处理回落至`2345`端口的流量，它其实是`vmess+TCP`：

```json
{
    "port": 2345,
    "listen": "127.0.0.1",
    "protocol": "vmess",
    "settings": {
        "clients": [
            {
                "id": "", // 填写你的 UUID
                "level": 0,
                "email": "love@example.com"
            }
        ]
    },
    "streamSettings": {
        "network": "tcp",
        "security": "none",
        "tcpSettings": {
            "acceptProxyProtocol": true,
            "header": {
                "type": "http",
                "request": {
                    "path": [
                        "/vmesstcp" // 必须换成自定义的 PATH，需要和分流的一致
                    ]
                }
            }
        }
    }
},
```

4、后续处理回落至`3456`端口的流量，它其实是是`vmess+ws(+cdn)`：

```json
{
    "port": 3456,
    "listen": "127.0.0.1",
    "protocol": "vmess",
    "settings": {
        "clients": [
            {
                "id": "", // 填写你的 UUID
                "level": 0,
                "email": "love@example.com"
            }
        ]
    },
    "streamSettings": {
        "network": "ws",
        "security": "none",
        "wsSettings": {
            "acceptProxyProtocol": true, // 提醒：若你用 Nginx/Caddy 等反代 WS，需要删掉这行
            "path": "/vmessws" // 必须换成自定义的 PATH，需要和分流的一致
        }
    }
}
```

至此，模板的完整回落路线为：

![](/images/vless3.png)

以上就是`Xray`的回落功能介绍。

