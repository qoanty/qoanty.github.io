---
title: "Debian 10进行初始服务器设置"
date: 2019-07-16T14:45:27+08:00
tags: ["SSH", "Debian"]
categories: ["Debian", "SSH"]
draft: false
---

## 介绍

首次创建新的Debian 10服务器时，您应在基本设置的早期阶段执行一些配置步骤，这将提高服务器的安全性和可用性，并为后续操作奠定坚实的基础。

在本教程中，我们将学习如何以`root`用户登录到服务器，创建具有管理员权限的新用户以及如何设置基本防火墙。

## 第1步 — 以root用户登录

要登录服务器，您需要知道服务器的公共`IP`地址以及密码，如果您安装了用于身份验证的`SSH`密钥，还需要`root`用户的私钥。如果您尚未连接到服务器，请输入以下命令以`root`用户登录（需替换为您的服务器`IP`地址）：

```bash
ssh root@your_server_ip
```

如果使用密码认证，请输入`root`密码登录。如果使用受密码保护的`SSH`密钥，则在每次会话中第一次使用密钥时，系统可能会提示您输入密码。如果这是您第一次使用密码登录服务器，则可能还会提示您更改`root`密码。

**关于Root用户**

在Linux系统中`root`用户具有非常广泛的权限，因此不建议您定期使用它，这是因为即使在偶然的情况下，`root`用户固有的部分权限也可以进行具有破坏性的更改能力。

下一步是设置备用用户，以减少对日常工作的影响。稍后，我们将解释如何在您需要的时候增加权限。

## 第2步 — 创建新用户

以`root`用户登录后，我们准备添加从现在开始将用于登录的新用户。

本示例创建一个名为`qoant`的新用户，您需要替换为您喜欢的用户名：

```bash
adduser qoant
```

从用户密码开始，系统将询问您几个问题。

输入一个强密码，并选择填写您想要的任何其他信息。这不是必需的，您可以单击`ENTER`键跳过任何字段。

接下来，我们将使用管理员权限设置此新用户。

## 第3步 — 授予管理权限

现在，我们创建了一个具有普通权限的新用户。但是，有时我们可能需要执行管理任务。

为了避免注销普通用户并以`root`用户重新登录，我们可以把普通用户设置成所谓的超级用户或具有`root`权限。这将使我们的普通用户通过在命令前加上`sudo`来以管理员权限运行。

要将这些权限添加到我们的新用户，我们需要将新用户添加到`sudo`组。Debian 10在默认情况下，允许属于`sudo`组的用户使用`sudo`命令。

以`root`用户运行此命令，将您的新用户添加到`sudo`组（需替换为您的新用户）：

```bash
usermod -aG sudo qoant
```

现在，以普通用户身份登录后，您可以在命令前输入`sudo`以超级用户权限运行该命令。

## 第4步 — 设置基本防火墙

Debian服务器可以使用防火墙来确保仅允许与特定服务的端口连接。在本教程中，我们将安装和使用`UFW`防火墙来帮助设置防火墙策略和管理异常。

我们可以使用`apt`软件包管理器来安装`UFW`。更新本地索引以检索有关可用软件包的最新信息，然后通过输入以下命令来安装`UFW`防火墙并进行基本设置：

```bash
apt update
apt install ufw
# 拒绝所有传入流量
ufw default deny incoming
# 允许所有传出流量
ufw default allow outgoing
```

防火墙配置文件允许`UFW`管理已安装应用程序命名的防火墙规则集。默认情况下，某些常用软件的配置文件与`UFW`捆绑在一起，并且软件包可以在安装过程中向`UFW`注册其配置文件。`OpenSSH`是一个允许我们连接到服务器的服务，它有一个可使用的防火墙配置文件。

您可以通过输入以下命令列出所有可用的应用程序配置文件：

```bash
ufw app list

Output
Available applications:
 . . .
 OpenSSH
 . . .
```

我们需要确保防火墙允许`SSH`连接，以便下次可以重新登录。输入以下命令：

```bash
# 允许SSH连接
ufw allow OpenSSH
# 启用防火墙
ufw enable
# 查看防火墙状态
ufw status

Output
Status: active

To             Action   From
--             ------   ----
OpenSSH          ALLOW    Anywhere
OpenSSH (v6)     ALLOW    Anywhere (v6)
```

由于防火墙当前阻止除`SSH`以外的所有连接，因此，如果安装和配置其他服务，则需要调整防火墙设置以允许可接受的流量进入。

## 第5步 — 为普通用户启用外部访问

现在我们有了一个日常使用的普通用户，我们需要确保可以通过`SSH`直接登录该帐户。

注意：在确认您是否可以登录并对新用户使用`sudo`之前，我们建议您保持以`root`用户登录。这样，如果遇到问题，您可以进行故障排除并以`root`用户进行任何必要的更改。

为新用户配置`SSH`访问的过程取决于服务器的`root`用户是使用密码还是`SSH`密钥进行身份验证。

### 如果Root用户使用密码验证

如果您使用密码登录到`root`用户，那么将为`SSH`启用密码身份验证。您可以通过打开新的终端会话并使用带有新用户名的`SSH `进行连接：

```bash
ssh qoant@your_server_ip
```

输入普通用户的密码后登录。请记住，如果您需要运行具有管理权限的命令，请像下面这样输入：

```bash
sudo command_to_run
```

每次会话第一次使用`sudo`时，系统都会提示您输入用户名及密码（之后还会定期提示）。

为了增强服务器的安全性，强烈建议您设置`SSH`密钥，而不要使用密码身份验证。

### 如果Root用户使用密钥验证

如果您使用`SSH`密钥登录到`root`用户，则将禁用`SSH`的密码身份验证。您需要将本地公共密钥的副本添加到新用户的`~/.ssh/authorized_keys`文件中才能成功登录。

由于您的公钥已经在服务器上`root`用户的`~/.ssh/authorized_keys`文件中，因此我们可以在现有会话中使用`cp`命令将该文件和目录结构复制到我们的新用户中。之后，我们可以使用`chown`命令更改文件的权限。

确保更改为您的普通用户名称：

```bash
cp -r ~/.ssh /home/qoant
chown -R qoant:qoant /home/qoant/.ssh
```

用`cp -r`命令将整个目录复制到新用户的主目录下，然后用`chown -R`命令将该目录（及其中的所有内容）的所有者更改为指定的`username:groupname`（默认情况下，Debian创建的用户名与组名相同）。

现在，打开一个新的终端会话，并使用您的新用户名通过`SSH`登录：

```bash
ssh qoant@your_server_ip
```

您应该在不使用密码的情况下登录到新用户。请记住，如果您需要运行具有管理权限的命令，请在命令前输入`sudo`，如下所示：

```bash
sudo command_to_run
```

每次会话第一次使用`sudo`时，系统都会提示您输入用户名及密码（之后还会定期提示）。

## 第6步 — 禁用Root用户登录

输入以下命令，找到`root`用户登录`SSH`的当前设置：

```bash
sudo cat /etc/ssh/sshd_config | grep PermitRootLogin
PermitRootLogin without-password
```

上面的输出结果表明已经设置为无密码，这意味着禁用了密码身份验证，而是启用了公钥身份验证。在大多数情况下都可以，确保这一行不应该被注释掉，或者不应该设置为`yes`。

要完全禁用`root`用户登录，请输入以下命令：

```bash
sudo vim /etc/ssh/sshd_config
# 更改为以下内容
PermitRootLogin no
# 保存并重新启动SSH服务
sudo systemctl restart sshd
sudo systemctl status sshd
```

现在，您将被禁止以`root`用户登录。

## 第7步 — 更改SSH默认端口

互联网上的恶意爬虫程序会持续针对`SSH`默认的22端口进行扫描。您可以将其更改为任何其他端口，以使您的服务器不会成为恶意爬虫程序继续攻击的对象。要更改`SSH`端口，请输入以下命令：

```bash
sudo vim /etc/ssh/sshd_config
# 找到该行
#Port 22
# 更改为下行
Port 7777
# 保存并设置防火墙开放该端口
sudo ufw allow 7777
# 设置防火墙禁止22端口
sudo ufw delete allow 22
sudo ufw delete allow ssh
# 重新加载防火墙规则
sudo ufw reload
# 重新启动SSH服务
sudo systemctl restart sshd
```

现在，如果您尝试在不指定端口的情况下从另一个终端登录，则将被禁止，输入以下命令进行登录：

```bash
ssh -p 7777 qoant@your_server_ip
```

## 第8步 — 设定时区及主机名

您可能希望服务器与您所在的时区相同。输入以下命令：

```bash
# 获取可用时区列表
timedatectl list-timezones
# 设置时区
sudo timedatectl set-timezone Asia/Shanghai
# 确认时区
timedatectl
```
要设置主机名，请输入以下命令：

```bash
# 检查现有的主机名
hostnamectl
# 设置主机名
sudo hostnamectl set-hostname host.debian.site
```

要在本地服务器中解析主机名，您需要编辑`/etc/hosts`文件并加入以下内容：

```bash
127.0.0.1    localhost host.debian.site
```

## 结论

在本教程中，我们学习了如何在新创建的Debian 10服务器上设置`sudo`用户，我们还保护了`SSH`服务不受基本入侵的影响。
