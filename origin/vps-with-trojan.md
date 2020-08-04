---
title: "Trojan安装配置教程"
date: 2019-04-30T19:37:11+08:00
tags: ["VPS", "Trojan"]
categories: ["Trojan", "VPS"]
draft: false
---

## 简介

[Trojan](https://github.com/trojan-gfw/trojan)是近两年兴起的网络工具，与强调加密和混淆的SS/SSR等工具不同，trojan将通信流量伪装成互联网上最常见的https流量，从而有效防止流量被GFW检测和干扰。

### v2ray和trojan的区别及特点：

- v2ray是一个网络框架，功能齐全；trojan只是一个绕过防火墙的工具，轻量级、功能简单；
- v2ray和trojan都能实现https流量伪装；
- v2ray内核用go语言开发，trojan是c++实现，理论上trojan比v2ray性能更好；
- v2ray使用的人多，客户端很好用；trojan关注和使用的人少，官方客户端简陋，生态完善度不高。

### Trojan工作原理

Trojan是一个比较新的翻墙软件，在设计时采用了更适应国情的思路。在穿透GFW时，人们认为强加密和随机混淆可能会欺骗GFW的过滤机制。然而，trojan实现了这个思路的反面：它模仿了互联网上最常见的https协议，以诱骗GFW认为它就是https，从而不被识别。 

![](/images/trojan.svg)

如图所示，trojan工作在443端口，并且处理来自外界的https请求，如果是合法的trojan请求，那么为该请求提供服务，否则将该流量转交给web服务Nginx，由Nginx为其提供服务。基于这个工作过程可以知道，trojan的一切表现均与Nginx一致，不会引入额外特征，从而达到无法识别的效果。当然，为了防止恶意探测，我们需要将80端口的流量全部重定向到443端口，并且服务器只开放80和443端口，这样可以使得服务器与常见的Web服务器表现一致。

## 先决条件

按照本教程部署trojan需要一台运行Linux的境外vps，一个解析到vps的域名以及为域名申请的一个证书，需通过SSH终端连接到vps。

## 创建新用户（可选）

为了系统安全，一般不建议直接使用`root`用户对系统做设置，而`VPS`默认只有`root`用户，因此可以自己新建一个具有`sudo`权限的用户继续后面的操作，代码如下所示，注意密码强度不能太低。第一条命令创建用户，第二条命令设置密码，第三条命令将该用户加入`sudo`组，第四条命令切换到该用户。

```bash
useradd -m -s /bin/bash qoant
passwd qoant
usermod -G sudo qoant
su -l qoant
```

## 创建服务用户

这里解释下为什么不直接使用`root`用户，因为`trojan`是占用443端口，使用`https`协议，所以必然是要安装证书的。本教程使用[acme.sh](https://github.com/Neilpang/acme.sh)为`trojan`生成证书，并配置了`acme.sh`的自动更新，包括代码和证书的更新。一方面，既然有配置自动更新就有可能出各种问题，毕竟你对更新之后的代码和证书是否可用一无所知。另一方面，`acme.sh`和`trojan`均为开源软件，不一定值得完全信赖。基于此，为了降低`acme.sh`和`trojan`对系统的影响和其相互影响，故需要单独为`acme.sh`和`trojan`建立没有`sudo`权限的用户。如果执意使用具有`sudo`权限的用户，或者直接使用`root`，出了问题是要自己承担后果的。

这里我们创建两个用户，分别为`trojan`和`acme`。其中用户`trojan`只需要运行`trojan`服务，无需登录，也无需家目录，故设置为系统用户即可。这里将用户`acme`也设置为系统用户，但是区别在于`acme`需要配置`acme.sh`，故需要家目录。注意到，我并未给用户`acme`设置密码，所以该用户也不能登录，只能通过其他已经登录的用户切换过去，这样尽可能的保证了系统的安全与任务的独立。因为`trojan`和`acme`都需要读写证书文件，所以将`acme`和`trojan`添加到同一个用户组`certusers`，待申请到证书后将证书所有权交给用户组`certusers`并允许组内用户访问即可。

```bash
sudo groupadd certusers
sudo useradd -r -M -G certusers trojan
sudo useradd -r -m -G certusers acme
```

## 配置Nginx

若你已经有配置`Nginx`虚拟主机，配置`Nginx`之前请，可以使用`cp`命令备份一下原来的虚拟主机，等介绍完基本配置再讲如何与现有服务集成。

```bash
sudo apt update
sudo apt install nginx
```
`Nginx`的虚拟主机配置文件在`/etc/nginx/sites-available/`文件夹中，如果要开启某一个虚拟主机，则建立一个软连接到`/etc/nginx/sites-enabled/`文件夹并重启`Nginx`即可。默认虚拟主机在`/etc/nginx/sites-enabled/`文件夹，需要关闭掉，否则会冲突。

使用如下命令关闭`Nginx`默认虚拟主机。

```bash
sudo rm /etc/nginx/sites-enabled/default
```

输入以下命令添加虚拟主机。（注意域名`<qoant.com>`改为你自己的域名，这是虚拟主机的文件名，只是用来自己识别的。）

```bash
sudo vim /etc/nginx/sites-available/<qoant.com>
```

基于`trojan`工作原理，现给定`Nginx`虚拟主机如下所示。这些内容可以直接拷贝到上面的配置文件中再修改为你自己的，其中要修改的地方为：

1. 第4行的`server_name`的值`<qoant.com>`改为你自己的域名；
2. 第7行的`proxy_pass`随便指向一个没有敏感信息的网站都可以，这就是你要反向代理的网站，这里是用的Debian地址；
3. 第15行的`server_name`的值`<8.8.8.8>`改为你自己的IP；
4. 第17行`<qoant.com>`改为自己的域名。

```text
server {
    listen 127.0.0.1:80 default_server;

    server_name <qoant.com>;

    location / {
        proxy_pass https://www.debian.org;
    }

}

server {
    listen 127.0.0.1:80;

    server_name <8.8.8.8>;

    return 301 https://<qoant.com>$request_uri;
}

server {
    listen 0.0.0.0:80;
    listen [::]:80;

    server_name _;

    location / {
        return 301 https://$host$request_uri;
    }

    location /.well-known/acme-challenge {
       root /var/www/acme-challenge;
    }
}
```

解释一下这些虚拟主机的一些细节：第一个`server`接收来自`trojan`的流量，与上面`trojan`配置文件对应；第二个`server`也是接收来自`trojan`的流量，但是这个流量尝试使用IP而不是域名访问服务器，所以将其认为是异常流量，并重定向到域名；第三个`server`接收除`127.0.0.1:80`外的所有80端口的流量并重定向到443端口，这样便开启了全站`https`，可有效的防止恶意探测，其中`.well-known`是为申请证书所准备的，下文用到的时候自然明白。注意到，第一个和第二个`server`对应原理图中的蓝色数据流，第三个`server`对应原理图中的红色数据流，原理图中的绿色数据流不会流到`Nginx`。

创建指向上述配置文件的符号链接并启用此虚拟主机，注意域名`<qoant.com>`改为你自己的域名。

```bash
sudo ln -s /etc/nginx/sites-available/<qoant.com> /etc/nginx/sites-enabled/
sudo systemctl restart nginx
sudo systemctl status nginx
```

## 配置证书

当从`Let’s Encrypt`获得证书时，`Let’s Encrypt`会验证证书中域名的控制权。一般采用`HTTP-01`或`DNS-01`方式来验证，详情参考官方文档[验证方式](https://letsencrypt.org/zh-cn/docs/challenge-types/)。本文使用`HTTP-01`方式验证，若需要使用`DNS-01`方式验证，参考`acme.sh`官方文档[`How to use DNS API`](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)。

### 创建证书文件夹

第一条命令新建一个文件夹`/etc/letsencrypt/live`用于存放证书。第二条命令将证书文件夹所有者改为`acme`，使得用户`acme`有权限写入证书。

```bash
sudo mkdir -p /etc/letsencrypt/live
sudo chown -R acme:acme /etc/letsencrypt/live
```

### 创建webroot并修改所有者

本文使用`acme.sh`的`http`方式申请证书，`http`方式需要在网站根目录下放置一个文件来验证域名所有权，故需要`acme.sh`和`nginx`均对`webroot`目录有权限，故将运行`Nginx`的`worker`进程加入`certusers`组，然后再将`webroot`目录附加给`certusers`组即可。

在不同的`Linux`发新版本中，`nginx`可能使用不同的用户运行`worker process`，可能为`www-data`，`nginx`，`nobody`中的一个，故需要自己运行以下命令查找`nginx: worker process`所属用户，然后根据实际情况，添加到`certusers`组： 

```bash
ps -eo user,command | grep nginx
sudo usermod -G certusers www-data
```

运行下面两条命令，第一条命令新建一个文件夹`/var/www/acme-challenge`用于给`acme.sh`存放域名验证文件。第二条命令将证书文件夹所有者改为`acme`，使得用户`acme`有权限写入文件，同时当验证的时候`Nginx`可以读取该文件。

```bash
sudo mkdir -p  /var/www/acme-challenge
sudo chown -R acme:certusers /var/www/acme-challenge
```

### 安装acme.sh自动管理CA证书脚本

分别执行如下命令，注意看是否报错。第一条命令切换到用户`acme`。第二条命令安装`acme.sh`。第三条命令退出当前用户。第四条命令再次切换到用户`acme`。注意到这里两次切换用户的操作不能省略，因为安装完`acme.sh`之后要重新登录当前用户，否则无法识别出`acme.sh`命令。

```bash
sudo su -l -s /bin/bash acme
curl https://get.acme.sh | sh
exit
sudo su -l -s /bin/bash acme
```

### 申请证书

执行如下命令（注意域名`<qoant.com>`改为你自己的域名），等待一会儿，会有证书申请成功的提示。

```bash
acme.sh --issue -d <qoant.com> -w /var/www/acme-challenge
```

**申请失败怎么办？** 证书申请失败的可能性一般有：1. 文件夹权限问题，请仔细检查每一步是否都正确；2. 证书申请次数超限，此时切忌反复尝试。证书每一个周申请次数是有限制的（20次），如果超限了就需要等一个周或者更换域名了（这个限制是争对每一个子域单独做的限制，所以万一超限了还可以用子域名继续部署）。解决方案是：在上述命令后加`--staging`参数继续测试。测试通过之后，删除生成的四个证书文件以及该域名对应的文件夹并取消`--staging`参数再执行一次。`--staging`参数申请的证书只作为测试用，客户端是无法认证通过的（提示`SSL handshake failed: tlsv1 alert unknown ca`），所以使用`--staging`参数申请到了证书之后要去掉`--staging`参数重新申请一次。

### 安装证书

执行如下命令（注意域名`<qoant.com>`改为你自己的域名），第一条命令使用`acme.sh`将证书安装到`certfiles`目录，这样`acme.sh`更新证书的时候会自动将新的证书安装到这里。第二条命令是配置`acme.sh`自动更新和自动更新证书，这样配置完`trojan`之后一般不用管服务器。

```bash
acme.sh --install-cert -d <qoant.com> \
	--key-file /etc/letsencrypt/live/private.key \
	--fullchain-file /etc/letsencrypt/live/certificate.crt
acme.sh --upgrade  --auto-upgrade
```

### 修改权限

最后还要允许组内用户访问证书。可通过如下命令实现。第一条命令将证书文件夹所在用户组改为`certusers`。第二条命令是赋予证书文件夹组内用户读取权限。运行这两条命令之后用户`trojan`就有权限读取证书了。第三条命令退出用户`acme`，因为证书已经安装完成。

```bash
chown -R acme:certusers /etc/letsencrypt/live
chmod -R 750 /etc/letsencrypt/live
exit
```

## 配置Trojan

### 安装trojan

分别执行如下四个命令，注意看是否报错。第一个命令是安装`trojan`，安装完成一般会提示版本号注意看是否是最新版本。第二个命令是将`trojan`配置文件的所有者修改为用户`trojan`，由于使用`sudo`安装的`trojan`，该配置文件默认是属于`root`用户的，而我们需要使用`trojan`用户运行`trojan`，不修改所有者会导致启动`trojan`遇到权限问题。第三个命令备份`trojan`配置文件，以防万一。第四个命令是使用`vim`修改配置文件。

```bash
sudo bash -c "$(curl -fsSL https://raw.githubusercontent.com/trojan-gfw/trojan-quickstart/master/trojan-quickstart.sh)"
sudo chown -R trojan:trojan /usr/local/etc/trojan
sudo cp /usr/local/etc/trojan/config.json /usr/local/etc/trojan/config.json.bak
sudo vim /usr/local/etc/trojan/config.json
```

第四个命令执行完之后屏幕会显示`trojan`的配置文件，定位到`password`、`cert`和`key`并修改。密码按自己喜好，`cert`和`key`分别改为`/etc/letsencrypt/live/certificate.crt`和`/etc/letsencrypt/live/private.key`，编辑完配置文件后保存退出。修改之后的`config`文件如下所示。另外，如果有`IPv6`地址，将`local_addr`的`0.0.0.0`改为`::`才可以使用。

```json
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password1",
        "password2"
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/etc/letsencrypt/live/certificate.crt",
        "key": "/etc/letsencrypt/live/private.key",
        "key_password": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": ""
    }
}
```

请重点关注配置文件中的如下参数：

1. `local_port`：监听的端口，默认是443，除非端口被墙，不建议改成其他端口；
2. `remote_addr`和`remote_port`：非`trojan`协议时，将请求转发处理的地址和端口。可以是任意有效的ip/域名和端口号，默认是本机和80端口；
3. `password`：密码，｀需要几个密码就填几行，最后一行结尾不能有逗号；
4. `cert`和`key`：域名的证书和密钥；
5. `key_password`：默认没有密码（如果证书文件有密码就要填上）；
6. `alpn`：建议填两行：`http/1.1`和`h2`，保持默认也没有问题。

### 启动trojan

打开`trojan.service`文件，并将用户修改为`trojan`，内容如下。

```text
[Unit]
Description=trojan
Documentation=https://trojan-gfw.github.io/trojan
After=network.target network-online.target nss-lookup.target

[Service]
Type=simple
StandardError=journal
User=trojan
ExecStart=/usr/local/bin/trojan /usr/local/etc/trojan/config.json
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

然后重新加载配置文件。

```bash
sudo systemctl daemon-reload
```

执行以下命令，赋予`trojan`监听1024以下端口的能力，使得`trojan`可以监听到443端口。这是由于我们使用非`root`用户启动`trojan`，但是Linux默认不允许非`root`用户启动的进程监听1024以下的端口，除非为每一个二进制文件显式声明。

```bash
sudo setcap CAP_NET_BIND_SERVICE=+eip /usr/local/bin/trojan
```

`trojan`启动、查看状态命令分别如下，第一条是启动`trojan`，第二条是查看`trojan`运行状态，第三条是检查`trojan`是否在运行。启动之后再查看一下状态，`trojan`显示`active (running)`即表示正常启动了。如果出现`fatal: config.json(n): invalid code sequence`错误，那么是你的配置文件第n行有错误，请检查。如果启动失败，还可以用`journalctl -f -u trojan`查看`systemd`的日志。

```bash
sudo systemctl restart trojan
sudo systemctl status trojan
sudo ss -lp | grep trojan
```

### 检查服务器配置

到这里服务器就配置完成了。此时你可以在浏览器里面输入你的网站看是否能够访问，如果你的网站可以访问了，那么就一切正常啦。

另外，基于以上考虑到的可能的恶意探测，可以验证一下以下情况是否正常。

1. 浏览器中使用`ip`访问：重定向到[`https://qoant.com`](https://qoant.com);
2. 浏览器中使用`https://ip`访问：重定向到[`https://qoant.com`](https://qoant.com)（跳转的时候浏览器可能提示不安全是正常的）;
3. 浏览器中使用`qoant.com`访问：重定向到[`https://qoant.com`](https://qoant.com)。

## 配置防火墙（可选）

只打开22、80和443端口可以有效的阻止外部恶意流量，降低服务器暴露的风险。此步骤非必须，而且自己有其他服务记得其他服务的端口也要处理。

本文以`ufw`为例配置防火墙，`ufw`是一个很好用的防火墙配置命令，可以简化操作，减少错误的发生。

执行以下命令即可成功配置防火墙。注意，如果`ssh`端口不是22那么第二行需要调整（将`ssh`改为端口号）。

```bash
sudo ufw disable
sudo ufw allow ssh
sudo ufw allow https
sudo ufw allow http
sudo ufw enable
sudo ufw status
```

## 与现有Nginx服务集成

如果你本机已经有`Nginx`服务，那么`Nginx`配置文件需要做适当修改以和现有服务兼容。

1. 在原服务与`trojan`使用同一个域名且原来是监听443端口的情况下，那么需要将你的`ssl`配置删除并将监听地址改为第一个`server`监听的地址`127.0.0.1:80`，然后直接用修改好的`server`代替上述配置文件中第一个`server`即可。这样`https`加密部分将会由`trojan`处理之后转发给`Nginx`而不是由`Nginx`处理，原来的服务对于客户端来说就没有变化。

2. 如果原来的服务与`trojan`使用不同的域名，建议是修改`trojan`与原来的服务使用同一个域名，如果非要使用不同的域名，请参考第3点。

3. 如果原来的服务就监听了多个域名，那么请参考[`ngx_stream_ssl_preread_module`](https://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html)修改`Nginx`的`sni`。

4. 如果原来的服务是监听80端口，想要继续监听80端口那么直接去除第三个`server`即可（将域名验证相关`location`与自己的虚拟主机组合，否则无法更新证书），如果要改为监听443端口参考第1点。

## 配置trojan客户端

打开`trojan`客户端的配置文件，更改`remote_addr`为你的域名，`password`为你配置服务端时设置的密码，其它保持不变，内容如下。

```json
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "qoant.com",
    "remote_port": 443,
    "password": [
        "fuckgfw"
    ],
    "log_level": 1,
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:AES128-SHA:AES256-SHA:DES-CBC3-SHA",
        "sni": "",
        "alpn": [
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "curves": ""
    },
    "tcp": {
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    }
}
```

浏览器配置好`SwitchyOmega`插件，运行`trojan`命令即可翻越长城。
