---
title: "在VPS上设置Debian 10服务器"
date: 2019-07-20T12:31:13+08:00
tags: ["VPS", "Debian"]
categories: ["Debian", "VPS"]
draft: false
---

本文假定（并强烈建议）仅使用密钥验证的ssh登录（建议使用4096位RSA加密）。

## 设置Root用户

如果已经设置好ssh公钥，先登录。

```bash
ssh root@YOUR_HOST
```

更改默认的编辑器，通常会预先安装vim，nano等。

```bash
update-alternatives --config editor
```

### 添加别名（可选）

```bash
editor .bashrc
# add those `alias` commands to that file or uncomment existing alias
source .bashrc
```

### 添加非root用户并开启sudo权限

这里以qoant用户为例，需替换为您自己的非root用户。

```bash
adduser qoant # add user, only password is needed
apt update
apt upgrade
apt install sudo
usermod -aG sudo qoant
```

### 设置非root用户使用ssh-key

#### 方法1：切换用户

**情况1：与root用户使用相同的ssh-key**

```bash
su - qoant
mkdir .ssh
chmod 700 .ssh
sudo cat /root/.ssh/authorized_keys > .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```

**情况2：与root用户使用不同的ssh-key**

```bash
su - qoant
mkdir .ssh
chmod 700 .ssh
editor .ssh/authorized_keys # copy and paste the key
chmod 600 .ssh/authorized_keys
```

### 方法2：保持root用户

**情况1：与root用户使用相同的ssh-key**

```bash
cp -r ~/.ssh /home/qoant
chown -R qoant:qoant /home/qoant/.ssh
chmod 700 /home/qoant/.ssh
chmod 600 /home/qoant/.ssh/authorized_keys
```

**情况2：与root用户使用不同的ssh-key**

```bash
mkdir /home/qoant/.ssh
editor /home/qoant/.ssh/authorized_keys # copy and paste the key
chown -R qoant:qoant /home/qoant/.ssh
chmod 700 /home/qoant/.ssh
chmod 600 /home/qoant/.ssh/authorized_keys
```

完成后使用exit命令或Ctrl+D注销。

## 保护服务器

现在，使用刚创建的非root用户登录。

```bash
ssh qoant@YOUR_HOST
```

仅在极少数情况下，我们才需要以root用户登录（这并不安全）。

### 设置非root用户

添加别名。

```bash
editor .bash_aliases
source .bashrc
```

#### 自定义bachrc（可选）

编辑~/.bashrc文件。

- 更改提示符：终端窗口中的焦点应该放在命令的输出上，而不是提示上。
- 将目录添加到PATH：请注意，某些发行版将~/bin自动添加到PATH中。

```bash
...
# customizations
force_color_prompt=yes
...
## customizations
my_color_prompt=$color_prompt
...
# customizations
if [ "$my_color_prompt" = yes ]; then
  PS1="\[\e[38;2;245;150;170m\]◟(*•᎑•*)ノ♡:\[\e[00m\]\[\e[34m\]\w\[\e[00m\]\[\e[32m\]\$\[\e[00m\] "
else
  PS1="♡◟(*•᎑•*):\w\\$ "
fi
unset my_color_prompt
...
# customizations
PATH=/docker/bin:$PATH
```

#### 自定义Vim（可选）

编辑~/.vimrc文件。

```bash
# add color scheme etc.
# Example: wget -P ~/.vim/colors/ https://raw.githubusercontent.com/NLKNguyen/papercolor-theme/master/colors/PaperColor.vim
# change vimrc
vim .vimrc
```

示例：

```text
" syntax
syntax on
 
" history : how many lines of history VIM has to remember
set history=2000
 
" filetype
filetype on
" " Enable filetype plugins
filetype plugin on
filetype indent on
 
" base
set nocompatible                " don't bother with vi compatibility
set autoread                    " reload files when changed on disk, i.e. via `git checkout`
 
set magic                       " For regular expressions turn magic on
set title                       " change the terminal's title
 
set novisualbell                " turn off visual bell
set noerrorbells                " don't beep
set visualbell t_vb=            " turn off error beep/flash
set t_vb=
set tm=500
 
" show location
" set cursorcolumn
set cursorline
 
" show
set ruler                       " show the current row and column
set number                      " show line numbers
set nowrap
set showcmd                     " display incomplete commands
set showmode                    " display current modes
set showmatch                   " jump to matches when entering parentheses
set matchtime=2                 " tenths of a second to show the matching parenthesis
 
" search
set hlsearch                    " highlight searches
set incsearch                   " do incremental searching, search as you type
set ignorecase                  " ignore case when searching
set smartcase                   " no ignorecase if Uppercase char present
 
" tab
set expandtab                   " expand tabs to spaces
set smarttab
set shiftround
" indent
set autoindent smartindent shiftround
set shiftwidth=4
set tabstop=4
set softtabstop=4                " insert mode tab and backspace use 4 spaces
 
" encoding
set encoding=utf-8
set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
set termencoding=utf-8
set ffs=unix,dos,mac
set formatoptions+=m
set formatoptions+=B
 
" select and compare
set selection=inclusive
set selectmode=mouse,key
 
set completeopt=longest,menu
set wildmenu                           " show a navigable menu for tab completion"
set wildmode=longest,list,full
set wildignore=*.o,*~,*.pyc,*.class

" disable cndent for yml
au FileType yaml setlocal nocindent
 
" ============================ theme and status line ============================
 
" theme
set background=light
colorscheme PaperColor
 
" set mark column color
hi! link SignColumn   LineNr
hi! link ShowMarksHLl DiffAdd
hi! link ShowMarksHLu DiffChange
 
" status line
set statusline=%<%f\%h%m%r%=%k[%{(&fenc==\"\")?&enc:&fenc}%{(&bomb?\",BOM\":\"\")}]\%-14.(%l,%c%V%)\ %P
set laststatus=2   " Always show the status line - use 2 lines for the status bar
```

使用Vundle管理vim插件（可选）。

```bash
sudo apt install git
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
vim +PluginInstall +qall
```

### 加强SSH

```bash
sudo editor /etc/ssh/sshd_config
```

更改配置：
- 更改默认的SSH端口
- 禁用root用户登录ssh，PermitRootLogin no
- 禁用密码登录ssh，PasswordAuthentication no

```text
Port <some-other-number>
PermitRootLogin no
PasswordAuthentication no
```

确保您不会将自己锁定在服务器之外并应用更改。

```bash
sudo systemctl restart sshd
```

现在注销，然后使用新设置重新登录。

### 添加apt源列表（可选）

如果要使用默认源中未包含的软件或较新版本。

```bash
sudo editor /etc/apt/sources.list
wget http://nginx.org/keys/nginx_signing.key
sudo apt-key add nginx_signing.key
wget https://www.dotdeb.org/dotdeb.gpg
sudo apt-key add dotdeb.gpg
sudo aptitude update
```

### 配置防火墙（nftables）

列出当前规则集：

```bash
nft list ruleset
```

编辑配置文件（位于/etc/nftables.conf）。

```text
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        # established/related
        ct state established,related accept

        # invalid
        ct state invalid drop

        # loopback
        iifname lo accept

        # ssh
        tcp dport 22 ct state new accept # change to your ssh port

        # icmp
        ip protocol icmp accept
        ip6 nexthdr ipv6-icmp accept

        # http(s)
        tcp dport {http, https} accept
        udp dport {http, https} accept

        # uncomment to enable log
        #log prefix "[nftables] Input Drop: " flags all counter drop
    }
    chain forward {
        # drop everything (if not a router)
        type filter hook forward priority 0; policy drop;

        # uncomment to enable log
        #log prefix "[nftables] Forward Drop: " flags all counter drop
    }
    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

启用nftables。

```bash
sudo systemctl enable nftables
sudo systemctl start nftables
sudo systemctl status nftables
```

希望nftable能更加易于使用（当前许多软件仍不支持它）。

### 安装fail2ban

简而言之，fail2ban将阻止某些ip过于频繁地访问服务器，此工具可以使攻击者更难破解。

请注意，如果有必要，可能需要配置fail2ban使用nftables而不使用iptables。

```bash
sudo apt install fail2ban
cd /etc/fail2ban
sudo cp fail2ban.conf fail2ban.local
sudo cp jail.conf jail.local

# edit the 2 local conf files accordingly
# can delete anything but customized configs (see example below)
sudo editor fail2ban.local
sudo editor jail.local

# restart to apply changes
sudo systemctl restart fail2ban
```

其中fail2ban.local内容为空，jail.local内容如下。

```text
[DEFAULT]
bantime = 100m

[sshd]
enabled = true
port    = <some-ssh-port>
```

### 安装Docker（可选）

```bash
# install docker
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"

# start on boot
sudo systemctl enable docker

# add current user to docker group (so that no `sudo` required)
sudo groupadd docker # group might already exist
sudo usermod -aG docker $USER
## log out and log back in to apply this change
```

### 安装kubectl（可选）

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubectl

# bash completion (optional)
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source <(kubectl completion bash) # load it for current shell

# add my own kubeconfig file
echo "KUBECONFIG=/PATH/TO/MY/KUBECONFIG/" >> ~/.bashrc
```
