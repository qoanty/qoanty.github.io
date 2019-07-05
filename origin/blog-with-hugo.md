---
title: "使用Hugo搭建个人博客"
date: 2019-04-08T14:02:04+08:00
tags: ["Hugo", "Github"]
categories: ["Blog", "Hugo"]
draft: false
---

## 什么是Hugo

引用一下Hugo官网的描述

> The world’s fastest framework for building websites.

> Hugo是一个非常受欢迎的、开源的静态网站生成工具，它速度快，扩展性强。

更多关于Hugo的介绍，请参考Hugo的[官网]( https://gohugo.io/) 。

## Hugo的安装

Hugo速度快，而且不用依赖一大堆东西，一个二进制文件就可以搞定。

**Github下载**

我们可以直接从[Github Release](https://github.com/gohugoio/hugo/releases)页面下载对应的二进制文件，然后把它放在系统环境PATH目录里即可使用。这个支持任何平台，根据自己的平台选择相应的二进制包即可。

**Mac平台**

Mac下Hugo提供了homebrew安装的方式，非常简便。

```bash
brew install hugo
```

**Debian and Ubuntu平台**

```bash
sudo apt-get install hugo
```

**Windows平台**

Windows下Hugo提供了Chocolatey方式的安装，通过如下命令即可。

```bash
choco install hugo -confirm
```

**验证安装**

安转完成后，打开终端，输入如下命令进行验证是否安装成功。

```bash
hugo version
```

如果没问题的话，会输出Hugo的版本号等一些信息。

## Hugo的使用

**创建站点**

```bash
hugo new site blog
```

**创建文章**

创建about页面

```bash
hugo new about.md
```

about.md自动生成到content目录下。

创建一篇文章

```bash
hugo new posts/first.md
```

**添加主题**

Hugo默认不带主题，需要在[主题网站](https://themes.gohugo.io/) 选择喜欢的主题，然后点击下载链接从GitHub下载到themes目录中。

```bash
git clone https://github.com/rujews/maupassant-hugo themes/maupassant
```

然后在Hugo的配置文件里config.toml(yaml, json)中进行如下配置，即可使用。

```bash
theme = "maupassant"
```

## 运行Hugo

在站点根目录下执行Hugo命令进行调试：

```bash
hugo server --theme=maupassant --buildDrafts
```
（注明：如果在config.toml中配置指定主题可以省略--theme参数。）

浏览器里打开`http://localhost:1313`就可查看新创建的站点了。

## 部署

将博客部署在`GitHub Pages`上，首先在GitHub上创建一个Repository，命名为`qoant.github.io`。

在站点根目录执行Hugo命令生成最终页面：

```bash
hugo --theme=maupassant --baseUrl="https://qoanty.github.io/"
```

如果一切顺利，所有静态页面都会生成到public目录，将pubilc目录里所有文件push到创建的Repository的master分支。

```bash
cd public
git init
git remote add origin https://github.com/qoanty/qoanty.github.io.git
git add -A
git commit -m "first commit"
git push -u origin master
```

浏览器里访问[`https://qoanty.github.io/`](https://qoanty.github.io/)
