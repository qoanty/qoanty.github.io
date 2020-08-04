---
title: "Python虚拟环境和包管理工具Pipenv"
date: 2019-05-15T08:53:24+08:00
tags: ["Python", "Pipenv"]
categories: ["Pipenv", "Python"]
draft: false
---

Pipenv是[Kenneth Reitz](https://pipenv.kennethreitz.org/en/latest/)的作品，[Python](http://python.org/)官方推荐的包管理工具，它能够有效管理python多个版本及各种包。

它主要有以下特性：

1. Pipenv集成了pip，virtualenv两者的功能，且完善了两者的一些缺陷。
2. Pipenv使用Pipfile和Pipfile.lock来管理包的依赖关系，方便查看。
3. 各个地方使用了哈希校验，无论安装还是卸载包都十分安全。
4. 通过加载.env文件简化开发工作流程。
5. 支持Python2和Python3，在各个平台的命令都是一样的。

## 安装Pipenv

Debian 10默认安装了Python 2.7及Python 3.7两个版本，这里以Python3为例，首先安装对应的pip3，然后使用pip3安装pipenv，建议使用用户模式安装，防止与系统python库产生影响。

```bash
sudo apt install python3-pip
pip3 install --user pipenv
pip3 install --user --upgrade pipenv
```

编辑`.bashrc`文件，将用户目录添加到环境变量`export PATH="$PATH:$HOME/bin:$HOME/.local/bin"`。

## 使用Pipenv

Pipenv在每个项目的基础上管理包的依赖关系，转到项目目录，然后运行：

```bash
mkdir myproject
cd myproject
pipenv install requests-html
```

Pipenv将会安装[requests-html](https://requests-html.kennethreitz.org/)库，并且生成`Pipfile`文件用于跟踪项目在需要重新安装时（与其它人共享项目时）所需的依赖库，`Pipfile.lock`文件是通过hash算法将包的名称和版本及依赖关系生成哈希值，可以保证包的完整性。

创建一个的`main.py`文件并使用它：

```python
import requests
response = requests.get('https://httpbin.org/ip')
print('Your IP is {0}'.format(response.json()['origin']))
```

确保脚本所需要的包都已安装，使用`pipenv run`命令运行：

```bash
pipenv run python main.py
Your IP is 8.8.8.8
```

也可以使用`pipenv shell`命令，进入虚拟环境的shell。

## 常用命令一览

- pipenv --where                 显示本地工程信息
- pipenv --venv                  显示虚拟环境信息
- pipenv --py                    显示Python解释器信息
- pipenv install                 创建虚拟环境
- pipenv shell                   激活虚拟环境
- pipenv install [package]       安装包并加入到Pipfile
- pipenv install [package] --dev 安装开发包和默认包
- pipenv uninstall [package]     卸载包并从Pipfile中移除
- pipenv uninstall --all         卸载所有包并从Pipfile中移除
- pipenv graph                   显示当前安装的包及其依赖
- pipenv lock                    生成lockfile
- pipenv check                   检查安全漏洞
- pipenv run python [pyfile]     运行py文件
- pipenv clean                   卸载Pipfile.lock中未指定的所有包
- pipenv sync                    安装Pipfile.lock中指定的所有包
- pipenv update                  升级所有的包及其依赖
- pipenv --rm                    删除虚拟环境

