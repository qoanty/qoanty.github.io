---
title: "Manjaro-KDE 安装与设置"
date: 2020-07-18T10:12:26+08:00
tags: ["Manjaro", "KDE"]
categories: ["KDE", "Manjaro"]
draft: false
---

## Manjaro 简介

[Manjaro](https://manjaro.org) 是一款基于 [Arch Linux](https://www.archlinux.org)，对用户友好的 Linux 发行版。在 Linux 社区，Arch Linux 的确是一个异常快速、强大、轻量级的发行版，它提供最新的、最全的软件。

Manjaro 由奥地利、法国和德国的爱好者共同开发，提供了 Arch Linux 操作系统的所有优点，同时注重用户友好性和可用性。 Manjaro 提供32位和64位版本，适合新手以及经验丰富的 Linux 用户。

Manjaro 与 Arch 有许多相同的功能，包括：

- “滚动发行”开发模式，可提供最新的系统，而无需安装新版本
- 可用 AUR

然而，Manjaro 拥有自己的一些额外的功能，包括：

- 简化、用户友好的安装过程
- 自动检测计算机的硬件（例如显卡）
- 为系统自动安装必要的软件（例如显卡驱动程序）
- 有自己的专用软件仓库，以确保提供完全测试过的稳定的软件包
- 轻松安装和使用多个内核

## 制作启动u盘及安装系统

去 [manjaro](https://manjaro.org/download/) 官网下载系统，官方提供三种桌面环境，目前觉得 KDE 最合心意，功能最多，用起来也最顺手，对新手也友好，推荐使用。

下载iso文件之后，使用 [rufus](https://rufus.ie) 将其写入到U盘，选择默认选项即可，然后点击开始

关闭 bios 中的 FastBoot 以及 SecureBoot，然后开机选择U盘启动，进入图形化的系统安装界面，按照提示设置安装就好了。要注意的是：

1. 安装引导时一定要选择EFI分区，挂载点为/boot/efi，不要格式化；
2. 在安装的时候最好断网。

如果一切顺利的话，重启后就应该能进入 KDE 的桌面环境了。

## 更换软件源

配置中国区镜像，启动终端，输入：

```bash
sudo pacman-mirrors -i -c China -m rank
```

在弹出的框中选一个最快的源，不建议使用 archlinuxcn 的源，因为并不一定兼容。

接着同步并更新系统：

```bash
sudo pacman -Syyu
```

**pacman 常用命令：**

```bash
sudo pacman -S package　 # 安装
sudo pacman -R package　 # 删除单个软件包，保留其已安装的依赖关系
sudo pacman -Rs package  # 删除指定软件包，及其所有依赖关系
sudo pacman -Ss package  # 查找软件包
sudo pacman -Qs package  # 搜索已安装的包
sudo pacman -Sc    # 清理未安装的包文件
sudo pacman -Scc   # 清理所有的缓存文件
sudo pacman -Syu　 # 升级所有软件包
sudo pacman -Syy   # 同步软件仓库
sudo pacman -Syyu  # 更新系统的所有软件包

```

## 安装软件

Yay 是用 Go 编写的 Arch Linux AUR 包管理工具，有些时候官方仓库没有你想要的软件，就需要通过 yay 来安装，有了 yay 以后就不用 sudo pacman 了。

```bash
sudo pacman -S yay
```

**yay 常用命令：**

```bash
yay -S package   # 从AUR安装软件包
yay -Rns package # 删除软件包
yay -Qi package # 检查安装的版本
yay -Syu  # 升级所有已安装的包
yay -Ps   # 打印系统统计信息
yay -c    # 清理不需要的依赖
```

### Vim 及输入法

```bash
yay -S vim fcitx-im kcm-fcitx fcitx-googlepinyin
```

在`~/.xprofile`文件（没有则需要新建）中加入：

```text
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

重启后输入法即可使用。

### git

```bash
git config --global user.name "yourname"
git config --global user.email "youremail@gamil.com"
git config --global http.proxy "socks5://127.0.0.1:1080"
git config --global https.proxy "socks5://127.0.0.1:1080"
```

### zsh

```bash
sudo pacman -S zsh
chsh -s /bin/zsh
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
# 高亮插件
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
# 自动命令
git clone git://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
# 补全插件
wget https://mimosa-pudica.net/src/incr-0.2.zsh
mkdir ~/.oh-my-zsh/custom/plugins/incr
mv incr-0.2.zsh ~/.oh-my-zsh/custom/plugins/incr
# 主题
git clone https://github.com/reobin/typewritten.git $ZSH_CUSTOM/themes/typewritten
ln -s "$ZSH_CUSTOM/themes/typewritten/typewritten.zsh-theme" "$ZSH_CUSTOM/themes/typewritten.zsh-theme"
```

编辑`~/.zshrc`文件，修改并加入以下内容：

```text
ZSH_THEME="typewritten"
plugins=(git zsh-syntax-highlighting zsh-autosuggestions) 
source $ZSH/oh-my-zsh.sh
source $ZSH/custom/plugins/incr/incr*.zsh
```

保存退出后，使修改的配置文件生效：

```bash
source ~/.zshrc
```

手动修改 konsole 的默认 bash 为 zsh。

### 中文字体

```bash
yay -S ttf-dejavu
yay -S wqy-bitmapfont wqy-microhei wqy-microhei-lite wqy-zenhei
yay -S adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts
```

### WPS

```bash
yay -S wps-office 
yay -S wps-office-mui-zh-cn
yay -S ttf-wps-fonts
```

### QQ 及微信

```bash
yay -S deepin-wine-tim
yay -S deepin-wine-wechat
```

安装过程若提示如下内容，则是因为软件签名验证无法通过，这种情况也经常出现，解决方法为修改 [PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD) 文件。

```text
==> 正在验证 source 文件，使用md5sums...
    deepin.com.wechat_2.6.8.65deepin0_i386.deb ... 通过
    WeChatSetup-2.9.5.41.exe ... 失败
    run.sh ... 通过
    reg.patch ... 通过
    shadow.exe ... 通过
==> 错误： 一个或多个文件没有通过有效性检查！
Error downloading sources: deepin-wine-wechat
```

进入`~/.cache/yay/deepin-wine-wechat`文件夹，编辑 PKGBUILD 文件，找到对应的验证部分，将里面的验证码修改为 SKIP ，内容如下：

```bash
source=("$_mirror/pool/non-free/d/deepin.com.wechat/deepin.com.wechat_${deepinwechatver}_i386.deb"
  "${wechat_installer}-${pkgver}.exe::https://dldir1.qq.com/weixin/Windows/${wechat_installer}.exe"
  "run.sh"
  "reg.patch"
  "shadow.exe")
md5sums=('fe31cf4f0f6186fc1c99adc1512f5305'
  'SKIP'
  '42b388b01db50af8b781b58bc6ac5414'
  'f264f961704f2aa1d480971b0e58617a'
  'd83f1c3845f28abd81cbfd215089d3d8')
```

保存退出后，运行`makepkg -sri`命令进行安装。

**makepkg 常用命令：**

```bash
makepkg     # 构建软件包
makepkg -s  # 自动安装必须的依赖
makepkg -i  # 安装构建的软件包
makepkg -r  # 删除不再需要的编译依赖
makepkg -c  # 清空残余的文件和目录
```

### 科学上网

因为使用的是 v2ray，所以推荐一个 v2ray 的图形化配置工具 [qv2ray](https://github.com/Qv2ray/Qv2ray)，同时也支持 trojan，可以通过插件来实现，详见[使用方法](https://qv2ray.github.io/getting-started/)。

```bash
yay -S qv2ray v2ray
```

### 终端代理

```bash
yay -S proxychains-ng
sudo vim /etc/proxychains.conf
# 最后加入
socket5 127.0.0.1 1080
vim ~/.zshrc
# 最后加入
alias gfw='proxychains4'
# 测试
gfw curl ip.sb
```

### 截图工具

```bash
yay -S flameshot
```

设置快捷键启动方式：
设置 -> 快捷键 -> 自定义快捷键 -> 编辑 -> 新建 -> 全局快捷键 -> 命令/URL
设置触发器：设置为你习惯的快捷键 -> 动作：命令/URL填写：/usr/bin/flameshot gui

### 双系统时间同步

按照 [Arch WiKi](https://wiki.archlinux.org/index.php/System_time_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 的建议，这里修改 Windows 系统的注册表，以管理员方式打开 PowerShell，输入

```powershell
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_QWORD /f
```

### 常用软件

```bash
yay -S netease-cloud-music # 网易云音乐
yay -S google-chrome  # 浏览器
yay -S foxitreader 	  # 福昕阅读器
yay -S typora 	   # md编辑器
yay -S calibre 	   # 电子书管理
yay -S xmind 	   # 思维导图
yay -S nutstore    # 坚果云
yay -S wiznote	   # 为知笔记
yay -S xdman 	   # 下载软件
yay -S gimp 	   # 图形编辑
yay -S pencil	   # 原型图绘制
yay -S sublime	   # 代码编辑
yay -S timeshift   # 备份、回滚系统
yay -S putty remmina freerdp   # 远程桌面
yay -S virtualbox virtualbox-host-dkms # 虚拟机
yay -S gvim hugo freefilesync
```

使用 Manjaro 必须要有的习惯：

1. 必须要做 timeshift，以防你哪天玩坏只能重装。
2. 每天要开机执行一遍 sudo pacman -Syyu 更新系统。

虽说 Manjaro 相对 Arch 应该稳定一点，但终究是滚动发行版，还是有滚挂的风险，防止滚挂的最好办法就是及时滚动更新，长时间不更新必挂。
