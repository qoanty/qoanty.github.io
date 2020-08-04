---
title: "在Debian 10上安装Nginx"
date: 2019-07-23T11:25:15+08:00
tags: ["Nginx", "Debian"]
categories: ["Debian", "Nginx"]
draft: false
---

## 介绍

[Nginx](https://nginx.org/)是世界上最受欢迎的Web服务之一，负责托管互联网上一些规模很大，流量很高的网站。在大多数情况下，它比Apache更具资源友好性，可以用作Web服务或反向代理。

在本教程中，我们将讨论如何在Debian 10服务器上安装`Nginx`。

## 准备

在开始本教程之前，您应该已经在服务器上配置了具有`sudo`权限的非`root`用户和一个激活的防火墙。您可以按照[《Debian 10进行初始服务器设置》](https://qoant.com/2019/07/initial-server-with-debian/)教程学习如何设置它们。

如果您有可用的帐户，请以非`root`用户身份登录。

## 第1步 – 安装Nginx

因为`Nginx`在Debian的默认包管理库中可用，所以可以使用`apt`包管理系统直接安装它。

由于这是我们在此会话中首次使用`apt`包管理系统，因此我们首先要更新本地包索引，以便我们可以访问最新的包列表。之后，我们就可以安装`nginx`：

```bash
sudo apt update
sudo apt install nginx
```

## 第2步 – 调整防火墙

在测试`Nginx`之前，需要调整防火墙软件以允许访问该服务。使用`ufw`命令，查看并获取应用程序配置文件的列表：

```bash
sudo ufw app list

Output
Available applications:
...
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
...
```

`Nginx`提供了三种配置文件：

- `Nginx Full`：此配置文件打开80端口（未加密的Web流量）和443端口（TLS/SSL加密的流量）
- `Nginx HTTP`：此配置文件仅打开80端口（未加密的Web流量）
- `Nginx HTTPS`：此配置文件仅打开443端口（TLS/SSL加密的流量）

建议您启用限制性最强的配置文件，该配置文件仍允许您配置流量。由于我们尚未在本教程中为我们的服务器配置SSL，因此我们只需要开放80端口。

输入以下命令启用此功能：

```bash
sudo ufw allow 'Nginx HTTP'
```

输入以下命令验证更改并获取输出信息：

```bash
sudo ufw status

Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

## 第3步 – 检查Web服务

安装结束后，Debian 10将启动`Nginx`，Web服务也应该已经启动并运行。可以通过`systemd`输入以下命令来确保服务正在运行：

```bash
systemctl status nginx

Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2019-07-03 12:52:54 UTC; 4min 23s ago
     Docs: man:nginx(8)
 Main PID: 3942 (nginx)
    Tasks: 3 (limit: 4719)
   Memory: 6.1M
   CGroup: /system.slice/nginx.service
           ├─3942 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           ├─3943 nginx: worker process
           └─3944 nginx: worker process
```

如上所示，该服务已成功启动。但是，测试它的最佳方法是从`Nginx`请求一个页面。

您可以通过输入服务器的IP地址来访问默认的`Nginx`页面，以确认它是否正常运行。如果您不知道服务器的IP地址，请尝试在服务器的命令提示符下输入以下命令：

```bash
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

将返回几行输出，您可以在Web浏览器中依次尝试以查看它们是否有效。

获得服务器的IP地址后，将其输入到浏览器的地址栏：

```bash
http://your_server_ip
```

您应该看到默认的Nginx页面，显示此页面表示服务器正常运行。

## 第4步 – 管理Nginx进程

现在您已启动并运行了Web服务，让我们学习一些基本的管理命令。

```bash
# 停止Web服务器
sudo systemctl stop nginx
# 启动Web服务器
sudo systemctl start nginx
# 重启Web服务
sudo systemctl restart nginx
# 更改配置文件后重新加载
sudo systemctl reload nginx
# 禁止服务随机启动
sudo systemctl disable nginx
# 设置服务随机启动
sudo systemctl enable nginx
```

## 第5步 – 设置服务块

使用`Nginx` Web服务时，服务块（类似于Apache中的虚拟主机）可用于封装配置详细信息，并在一台服务器上托管多个域名。我们将设置一个名为**your_domain**的域名，您需要将其替换为自己的域名。

Debian 10上的`Nginx`默认启用了一个服务块，配置为`/var/www/html`目录中的文件。尽管这对于单个站点非常有效，但是如果托管多个站点，则可能变得难以管理。我们不修改`/var/www/html`目录，而是在`/var/www`目录下为**your_domain**站点创建一个目录结构，如果客户端请求与其他任何站点都不匹配，则将`/var/www/html`保留为要提供的默认目录。

输入以下命令创建**your_domain**目录并设置：

```bash
# 创建目录结构
sudo mkdir -p /var/www/your_domain/html
# 使用$USER环境变量分配目录的所有权
sudo chown -R $USER:$USER /var/www/your_domain/html
# 更改Web根目录的权限
sudo chmod -R 755 /var/www/your_domain
# 创建示例页面
vim /var/www/your_domain/html/index.html
```

添加以下示例HTML：

```html
<html>
    <head>
        <title>Welcome to your_domain</title>
    </head>
    <body>
        <h1>Success! Your Nginx server is successfully configured for <em>your_domain</em>. </h1>
<p>This is a sample page.</p>
    </body>
</html>
```

完成后保存并关闭文件。

为了使`Nginx`能够提供此内容，我们需要创建一个服务块，并正确指向我们自定义的Web目录。与直接修改默认的配置文件不同，我们在`/etc/nginx/sites-available/your_domain`目录下创建一个新文件：

```bash
sudo vim /etc/nginx/sites-available/your_domain
```

粘贴以下配置块内容，该配置块与默认配置块相似，但针对我们的新目录和域名进行了更新：

```text
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

请注意，我们已经将`root`配置更新到新目录，将`server_name`更新到我们的域名。

接下来，让我们通过在`sites-enabled`目录内创建指向我们自定义配置文件的符号链接来启用此服务块，`Nginx`在启动时从中读取该文件：

```bash
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

现在已启用并配置了两个服务块，并将其配置为根据`listen`和`server_name`指令响应请求：

- `your_domain`：将响应对`your_domain`和`www.your_domain`的请求。
- `default`：将响应80端口上与其他两个模块不匹配的任何请求。

为避免在配置中添加其他服务名称而可能引起的哈希存储区内存问题，需要调整`/etc/nginx/nginx.conf`文件中的单个值。打开文件：

```bash
sudo vim /etc/nginx/nginx.conf
```

找到`server_names_hash_bucket_size`指令并删除`#`符号以取消注释该行：

```text
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...
```

完成后保存并关闭文件。接下来，测试以确保`Nginx`文件中没有语法错误：

```bash
sudo nginx -t

Output
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

配置测试通过后，重新启动`Nginx`以启用您的更改：

```bash
sudo systemctl restart nginx
```

`Nginx`现在应该为您的域名服务，您可以通过在地址栏输入`http://your_domain`进行测试。

## 第6步 – 熟悉重要的Nginx文件和目录

既然您已知道如何管理`Nginx`服务，那么您应该花点时间熟悉一些重要的目录和文件。

### 内容

- `/var/www/html`：实际的Web内容（默认情况下仅由您之前看到的默认`Nginx`页面组成）从`/var/www/html`目录中提供。可以通过更改`Nginx`配置文件来更改。

### 服务配置

- `/etc/nginx`：`Nginx`配置目录。所有`Nginx`配置文件都位于此处。
- `/etc/nginx/nginx.conf`：主要的`Nginx`配置文件。可以对其进行修改以更改`Nginx`全局配置。
- `/etc/nginx/sites-available/`：可以存储每个站点服务块的目录。`Nginx`不会使用此目录中的配置文件，除非它们链接到`sites-enabled`目录。通常，所有服务块配置都在此目录中完成，然后通过链接到另一个目录来启用。
- `/etc/nginx/sites-enabled/`：存储已启用的每个站点服务块的目录。通常，通过链接到`sites-available`目录中的配置文件来创建这些文件。
- `/etc/nginx/snippets`：此目录包含一些配置片段，这些片段可以包含在`Nginx`配置的其他位置。潜在的可重复配置段是重构代码段的良好候选者。

### 服务日志

- `/var/log/nginx/access.log`：除非将`Nginx`配置为执行其他操作，否则对Web服务的每个请求都记录在此日志文件中。
- `/var/log/nginx/error.log`：任何`Nginx`错误都将记录在此日志中。

## 结论

现在，您已经安装了Web服务器，对于可以提供的内容类型以及可以用来为用户创造更丰富的体验的技术，您有很多选择。
