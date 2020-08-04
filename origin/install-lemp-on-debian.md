---
title: "在Debian 10上安装Linux，Nginx，MariaDB，PHP"
date: 2019-07-28T18:22:37+08:00
tags: ["Nginx", "Debian"]
categories: ["Debian", "Nginx"]
draft: false
---

## 介绍

`LEMP`软件包是一组可用于提供动态网页和Web应用程序的软件。“LEMP”是一个首字母缩略词，用于描述`Linux`操作系统和`(E)Nginx`网络服务，后端数据存储在`MariaDB`数据库中，动态处理由`PHP`处理。

尽管此软件包通常包括`MySQL`作为数据库管理系统，但某些`Linux`发行版（包括Debian）使用`MariaDB`替代`MySQL`。

在本教程中，您将使用`MariaDB`作为数据库管理系统，在Debian 10服务器上安装`LEMP`软件包。

## 先决条件

要完成本教程，您将需要访问Debian 10服务器，该服务器应配置了具有`sudo`权限的普通用户和启用了`ufw`防火墙，可以参考[《Debian 10进行初始服务器设置》](https://qoant.com/2019/07/initial-server-with-debian/)教程。

## 第1步 – 安装Nginx Web服务

为了向您的网站访问者提供网页服务，我们将使用`Nginx`，这是一款流行的Web服务，以其整体性能和稳定性而闻名。

您将使用的所有软件都直接来自Debian的默认软件包存储库。这意味着您可以使用`apt`软件包管理套件来完成安装。

由于这是您第一次使用`apt`，因此应首先更新本地软件包索引，然后再安装服务：

```bash
sudo apt update
sudo apt install nginx
```

在Debian 10上，`Nginx`配置为在安装后开始运行。

如果您正在运行`ufw`防火墙，则需要允许连接到`Nginx`。您应该启用限制最严格的配置文件，该配置文件仍将允许您所需的流量。由于您还没有为服务器配置`SSL`，因此现在您只需要允许`80`端口上的`HTTP`通信。

您可以通过输入以下命令启用此功能：

```bash
sudo ufw allow 'Nginx HTTP'
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

现在，通过在Web浏览器中访问服务器的域名或公共IP地址来测试服务是否已启动并正在运行。如果您没有指向服务器的域名，并且您不知道服务器的公共IP地址，则可以通过在终端中输入以下命令来找到它：

```bash
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

这将输出一些IP地址，您可以依次在Web浏览器中尝试输入它们。

在Web浏览器中输入IP地址，您将看到`Nginx`的默认页面：

```bash
http://your_domain_or_IP
```

如果能看到`Nginx`默认页面，则说明您已经成功安装了`Nginx`。

## 第2步 – 安装MariaDB

现在，您已经启动并运行了Web服务，您需要安装数据库系统，以便能够存储和管理站点数据。

在Debian 10中，传统上用于安装`MySQL`服务的`mysql-server`包替换为`default-mysql-server`包，这个包引用了`MariaDB`，它是`Oracle`原始`MySQL`服务的一个社区分支，目前是基于debian的软件包管理器存储库中提供的默认`MySQL`兼容数据库服务。

但是，为了获得长期兼容性，建议您不要使用该软件包，而应使用程序的实际`mariadb-server`包来安装`MariaDB`。

输入以下命令进行安装：
	
```bash
sudo apt install mariadb-server
```

安装完成后，建议您运行与`MariaDB`一起预装的安全脚本。该脚本将删除一些不安全的默认设置，并锁定对数据库系统的访问。输入以下命令启动交互式脚本：

```bash
sudo mysql_secure_installation
```

该脚本将引导您完成一系列提示，您可以在其中对`MariaDB`设置进行一些更改。第一个提示将要求您输入当前数据库的`root`密码，请不要与系统`root`目录混淆。该数据库`root`用户是对数据库系统具有完全权限的管理用户。因为您刚刚安装了`MariaDB`且还没有进行任何配置更改，所以该密码为空，因此只需在提示下按`ENTER`键即可。

下一个提示将询问您是否要设置数据库`root`密码。由于`MariaDB`对`root`用户使用了一种特殊的身份验证方法，该方法通常比使用密码更安全，所以现在不需要设置它，键入`N`，然后按`ENTER`键。

然后，您可以按`Y`，然后按`ENTER`键接受所有后续问题的默认设置。这将删除匿名用户和测试数据库，禁用远程`root`登录，并加载这些新规则，以便`MariaDB`立即执行您所做的更改。

完成后，输入以下命令登录到`MariaDB`控制台：

```bash
sudo mariadb
```

这将以管理数据库的`root`用户连接到`MariaDB`服务，这是在运行此命令时使用`sudo`命令实现的。您应该看到如下输出：

```bash
Output

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 74
Server version: 10.3.15-MariaDB-1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

请注意，您无需提供密码即可以`root`用户进行连接，这是因为管理`MariaDB`用户的默认身份验证方法是`unix_socket`而不是密码。尽管这看起来像是一个安全问题，但它使数据库服务更加安全，因为只允许以`root`用户登录`MariaDB`的用户是具有`sudo`权限的系统用户，这些用户通过控制台或运行具有相同权限的应用程序进行连接。实际上，这意味着您将无法使用管理数据库的`root`用户从`PHP`应用程序进行连接。

为了提高安全性，最好为每个数据库设置专用的用户，并为其设置较少的扩展权限，尤其是当您计划在服务器上托管多个数据库时。为了演示这种设置，我们将创建一个名为`example_database`的数据库和一个名为`example_user`的用户，但是您可以用不同的名称来进行替换。

要创建新数据库，请从您的`MariaDB`控制台输入以下命令：

```bash
MariaDB [(none)]> CREATE DATABASE example_database;
```

现在，您可以创建一个新用户，并授予他对刚创建的自定义数据库的全部权限。以下命令将该用户的密码定义为`password`，但是您应该使用自己选择的安全密码替换该值。

```bash
MariaDB [(none)]> GRANT ALL ON example_database.* TO 'example_user'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;
```

这将为`example_user`用户提供对`example_database`数据库的全部权限，同时阻止该用户在服务器上创建或修改其他数据库。

更新权限以确保在当前会话中保存并可用，然后退出：

```bash
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> exit
```

您可以使用自定义用户再次登录到`MariaDB`控制台，以测试新用户是否具有适当的权限：

```bash
mariadb -u example_user -p
```

注意此命令中的`-p`选项，它将提示您输入创建`example_user`用户时使用的密码。登录到`MariaDB`控制台后，请确认您有权访问`example_database`数据库：

```bash
MariaDB [(none)]> SHOW DATABASES;

Output
+--------------------+
| Database           |
+--------------------+
| example_database   |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)

exit
```

至此，您的数据库系统已经建立，您可以继续安装`PHP`，它是`LEMP`包的最后组件。

## 第3步 – 安装PHP进行处理

您已经安装了`Nginx`来提供内容，并安装了`MySQL`来存储和管理数据。现在，您可以安装`PHP`来处理代码并为Web服务生成动态内容。

虽然`Apache`在每个请求中都嵌入了`PHP`解释器，但`Nginx`需要一个外部程序来处理`PHP`并充当`PHP`解释器本身和Web服务之间的桥梁。这可以在大多数基于`PHP`的网站中提供更好的整体性能，但是需要其他配置。您需要安装`php-fpm`，它代表“PHP fastCGI进程管理器”，并告诉`Nginx`将`PHP`请求传递给该软件进行处理。此外，还需要`php-mysql`模块，该模块允许`PHP`与基于`MySQL`的数据库进行通信。核心`PHP`软件包将自动作为依赖项安装。

要安装`php-fpm`和`php-mysql`软件包，请运行：

```bash
sudo apt install php-fpm php-mysql
```

现在，您已经安装了`PHP`组件。接下来，您将配置`Nginx`以使用它们。

## 第4步 – 配置Nginx使用PHP处理器

使用`Nginx`服务时，可以使用用于封装配置详细信息的服务块（类似于`Apache`中的虚拟主机），并在一台服务器上托管多个域。在本教程中，我们将使用`your_domain`作为示例域名。

在Debian 10上，`Nginx`默认情况下启用了一个服务块，并配置为在`/var/www/html`目录中提供文档。尽管这对于单个站点来说效果很好，但是如果您托管多个站点，则可能变得难以管理。我们不必修改`/var/www/html`，而是在`/var/www`中为`your_domain`网站创建一个目录结构，如果客户端请求与任何其他站点都不匹配，则保留`/var/www/html`作为默认目录。

为`your_domain`创建Web目录，如下所示：

```bash
sudo mkdir /var/www/your_domain
```

接下来，使用`$USER`环境变量分配目录的所有权，该环境变量引用当前的系统用户：

```bash
sudo chown -R $USER:$USER /var/www/your_domain
```

然后，使用编辑器在`Nginx`的`sites-available`目录中新建一个配置文件：

```bash
sudo vim /etc/nginx/sites-available/your_domain
```

这将创建一个新的空白文件，粘贴以下基本配置：

```text
server {
    listen 80;
    listen [::]:80;

    root /var/www/your_domain;
    index index.php index.html index.htm;

    server_name your_domain;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
    }
}
```

这是一个基本配置，它监听80端口并为刚创建的Web目录中的文件提供服务。它只会响应`server_name`提供的域名或IP地址的请求，并且所有以`.php`结尾的文件将由`php-fpm`在`Nginx`将结果发送给用户之前进行处理。

完成编辑后，保存并关闭文件。通过链接到`Nginx`的`sites-enabled`目录中的配置文件来激活您的配置：

```bash
sudo ln -s /etc/nginx/sites-available/your_domain/etc/nginx/sites-enabled/
```

这将告诉`Nginx`在下一次重新加载时使用配置。您可以通过输入以下命令来测试配置是否存在语法错误：

```bash
sudo nginx -t
```

如果报告了任何错误，请返回配置文件以查看其内容，然后再继续。

准备好后，重新加载`Nginx`以使更改生效：

```bash
sudo systemctl reload nginx
```

接下来，您将在新的Web目录中创建一个文件以测试`PHP`处理。

## 第5步 – 创建一个PHP文件以测试配置

您的`LEMP`包现在应该已全部设置好了。您可以对其进行测试，以验证`Nginx`是否可以将`.php`文件正确地传递给`PHP`处理器。

您可以通过在文档目录中创建一个测试`PHP`文件来实现。在文档目录中用文本编辑器新建一个`info.php`文件：

```bash
vim /var/www/your_domain/info.php
```

将以下内容输入或粘贴到新文件中。这是有效的`PHP`代码，它将返回有关您的服务器的信息：

```php
<?php
phpinfo();
?>
```

完成后，保存并关闭文件。现在，您可以在Web浏览器中访问此页面，方法是访问您在`Nginx`配置文件中设置的域名或公共IP地址，然后输入`/info.php`：

```bash
http://your_domain/info.php
```

您将看到一个包含有关服务器的详细信息的网页。在通过该页面检查有关`PHP`服务器的相关信息之后，最好删除您创建的文件，因为该文件包含有关`PHP`环境和Debian服务器的敏感信息。您可以使用`rm`命令来删除该文件：

```bash
rm /var/www/your_domain/info.php
```

如果以后需要，可以随时重新生成该文件。接下来，我们将从`PHP`端测试数据库连接。

## 第6步 – 从PHP测试数据库连接（可选）

如果要测试`PHP`是否能够连接到`MariaDB`并执行数据库查询，那么可以创建一个包含虚拟数据的测试表，并从`PHP`脚本中查询其内容。

首先，使用您在本教程第2步中创建的数据库用户连接到`MariaDB`控制台：

```bash
mariadb -u example_user -p
```

创建一个名为`todo_list`的表，在`MariaDB`控制台中，输入以下命令：

```bash
CREATE TABLE example_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id)
);
```

现在，在测试表中插入几行内容。您可能需要使用不同的值重复执行这个命令几次：

```bash
MariaDB [(none)]> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
```

要确认数据已成功保存到表中，请运行：

```bash
MariaDB [(none)]> SELECT * FROM example_database.todo_list;
```

您将看到如下输出：

```bash
Output
+---------+--------------------------+
| item_id | content                  |
+---------+--------------------------+
|       1 | My first important item  |
|       2 | My second important item |
|       3 | My third important item  |
|       4 | and this one more thing  |
+---------+--------------------------+
4 rows in set (0.000 sec)
```

确认测试表中的有效数据后，可以退出`MariaDB`控制台：

```bash
MariaDB [(none)]> exit
```

现在您可以创建`PHP`脚本，该脚本将连接到`MariaDB`并查询您的内容。使用文本编辑器在自定义Web目录中创建一个新的`PHP`文件：

```bash
vim /var/www/your_domain/todo_list.php
```

将以下内容添加到您的`PHP`脚本中：

```php
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```

完成编辑后，保存并关闭文件。

现在，您可以在Web浏览器中访问此页面，方法是访问您在`Nginx`配置文件中设置的域名或公共IP地址，然后输入`/todo_list.php`：

```bash
http://your_domain/todo_list.php
```

您应该看到显示您已插入测试表中的内容的页面，这意味着您的`PHP`环境已准备就绪，可以连接`MariaDB`服务并与之交互。

## 结论

在本教程中，您使用`Nginx`作为Web服务，为访问者提供`PHP`网站和应用程序建立了灵活的基础。您已经将`Nginx`设置为通过`php-fpm`处理`PHP`请求，还设置了`MariaDB`数据库来存储您的网站数据。

为了进一步改进当前的设置，您可以在`PHP`中安装`Composer`进行依赖关系和程序包管理，还可以使用`Let's Encrypt`为您的网站安装`OpenSSL`证书。

