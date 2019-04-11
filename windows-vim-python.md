# windows 10下使用vim8配置python3开发环境

## 安装Python与Vim

从[Python官网](https://www.python.org)下载[64位Python](https://www.python.org/downloads)并安装，从[Vim网站](https://www.vim.org)下载[64位Vim](https://github.com/vim/vim-win32-installer/releases)并安装。

运行`vim --version`，如果列表中有+python/dyn和+python3/dyn，
则Vim编辑器支持Python。在Vim编辑器中运行`:python3 import sys; print(sys.version)`，会输出编辑器当前的Python版本，如果报错，则编辑器不支持Python语言，需要重装或重新编译。

## 安装Chocolatey

Windows的包管理器[Chocolatey](https://chocolatey.org)，以管理员权限打开命令提示符窗口，运行以下命令。

```cmd
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

查看版本信息`choco -v`，升级`choco upgrade chocolatey`，搜索软件包`choco search package`，除了在命令行中搜索软件包，还可以直接在网站上搜索[软件包](https://chocolatey.org/packages)。

## 安装Vundle

[Vundle](https://github.com/VundleVim/Vundle.vim)是一个Vim的插件管理器，使用它可以方便的安装其它插件，Windows下需要依赖[Git](https://git-scm.com)和[Curl](https://curl.haxx.se)软件。

### 安装Git和Curl

```cmd
C:\> choco install -y git
C:\> choco install -y curl
```

查看版本信息确认正确安装。

```cmd
C:\> git --version
git version 2.21.0.windows.1
C:\> curl --version
curl 7.64.0 (x86_64-pc-win32) libcurl/7.64.0 OpenSSL/1.1.1a (Schannel) zlib/1.2.11 brotli/1.0.7 WinIDN libssh2/1.8.0 nghttp2/1.36.0
Release-Date: 2019-02-06
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp
Features: AsynchDNS IDN IPv6 Largefile SSPI Kerberos SPNEGO NTLM SSL libz brotli TLS-SRP HTTP2 HTTPS-proxy MultiSSL
```

### Windows下安装及配置Vundle

打开命令提示符窗口，运行以下命令。

```cmd
cd %USERPROFILE%
md .vim\bundle
git clone https://github.com/VundleVim/Vundle.vim.git %USERPROFILE%/.vim/bundle/Vundle.vim
gvim .vimrc
```

将以下内容添加在`.vimrc`文件中。

```vim
set nocompatible              " 去除VI一致性,必须
filetype off                  " 必须

" 设置包括vundle和初始化相关的runtime path
set rtp+=$HOME/.vim/bundle/Vundle.vim
call vundle#begin('$HOME/.vim/bundle/')

" 让vundle管理插件版本,必须
Plugin 'VundleVim/Vundle.vim'

" 以下范例用来支持不同格式的插件安装
" 请将安装插件的命令放在vundle#begin和vundle#end之间
" Github上的插件
Plugin 'tpope/vim-fugitive'
" 来自 http://vim-scripts.org/vim/scripts.html 的插件
" Plugin 'L9'
" 不在Github上的插件
Plugin 'git://git.wincent.com/command-t.git'
" 本地的插件(例如自己的插件)
" Plugin 'file:///home/gmarik/path/to/plugin'
" 插件sparkup在仓库的子目录中
" 正确指定路径用以设置runtimepath
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" 如果已经安装了L9这个插件，可利用以下格式避免命名冲突
" Plugin 'ascenator/L9', {'name': 'newL9'}

" 你的所有插件需要在下面这行之前
call vundle#end()            " 必须
filetype plugin indent on    " 必须 加载vim自带和插件相应的语法和文件类型相关脚本
" 忽视插件改变缩进,可以使用以下替代:
"filetype plugin on
"
" 简要帮助文档
" :PluginList       - 列出所有已配置的插件
" :PluginInstall    - 安装插件,追加 `!` 用以更新或使用 :PluginUpdate
" :PluginSearch foo - 搜索 foo ; 追加 `!` 清除本地缓存
" :PluginClean      - 清除未使用插件,需要确认; 追加 `!` 自动批准移除未使用插件
"
" 查阅 :h vundle 获取更多细节和wiki以及FAQ
" 将你自己对非插件内容放在这行之后
```

设置完后可以在配置文件中添加要安装的插件，然后打开`vim`运行`:PluginInstall`或通过命令行直接运行`vim +PluginInstall +qall`进行安装。

## 开始打造IDE

### 分割窗口(Split Layouts)

使用`:sv <filename>`命令打开一个文件，可以纵向分割布局（新文件会在当前文件下方界面打开），使用相反的命令`:vs <filename>`，可以得到横向分割布局（新文件会在当前文件右侧界面打开），若要在指定屏幕上进行分割布局的区域，在`.vimrc`文件中添加下面代码。

```vim
set splitbelow
set splitright
```

不使用鼠标而通过快捷键切换分割布局，在`.vimrc`文件中添加下面代码。

```vim
"split navigations
nnoremap <C-J> <C-W><C-J>
nnoremap <C-K> <C-W><C-K>
nnoremap <C-L> <C-W><C-L>
nnoremap <C-H> <C-W><C-H>
```

快捷键

- <kbd>Ctrl</kbd> <kbd>J</kbd> 切换到下方的分割窗口
- <kbd>Ctrl</kbd> <kbd>K</kbd> 切换到上方的分割窗口
- <kbd>Ctrl</kbd> <kbd>L</kbd> 切换到右侧的分割窗口
- <kbd>Ctrl</kbd> <kbd>H</kbd> 切换到左侧的分割窗口

### 缓冲区(Buffers)

Vim提供了方便访问缓冲区的方式，只需要输入`:b <buffer name or number>`，就可以切换到一个已经开启的缓冲区（此处也可使用自动补全功能），还可以通过`:ls`命令查看所有的缓冲区。

### 代码折叠(Code Folding)

开启代码折叠并使用空格键控制，在`.vimrc`文件中添加下面代码。

```vim
" Enable folding
set foldmethod=indent
set foldlevel=99
" Enable folding with the spacebar
nnoremap <space> za
```

隐藏当前不需要关注的代码，`set foldmethod=ident`会根据每行的缩进开启折叠。但是这样做会出现超过所希望的折叠数目，可以通过[SimplyFold](https://github.com/tmhedberg/SimpylFold)插件解决这个问题，在`.vimrc`文件中加入下面代码。

```vim
Plugin 'tmhedberg/SimpylFold'
```

希望看到折叠代码的文档字符串。

```vim
let g:SimpylFold_docstring_preview=1
```

### 代码缩进(Python Indentation)

Vim中的缩进要能做到以下两点：

- 首先，缩进要符合PEP8标准。
- 其次，更好地处理自动缩进。

#### PEP8

要支持PEP8风格的缩进，在`.vimrc`文件中加入下面代码。

```vim
au BufNewFile,BufRead *.py,*.sh
    \ set tabstop=4	|
    \ set softtabstop=4 |
    \ set shiftwidth=4	|
    \ set textwidth=79	|
    \ set expandtab	|
    \ set autoindent	|
    \ set fileformat=unix
```

还可以针对每种文件类型进行设置。

```vim
au BufNewFile,BufRead *.js,*.html,*.css
    \ set tabstop=2	|
    \ set softtabstop=2 |
    \ set shiftwidth=2
```

#### 自动缩进

自动缩进在某些情况下（比如函数定义有多行的时候），并不总是会达到预想的效果，尤其是在符合PEP8标准方面，可以用[indentpython.vim](https://github.com/vim-scripts/indentpython.vim)插件来解决。

```vim
Plugin 'vim-scripts/indentpython.vim'
```

### 标示不必要的空白字符

让Vim标示出来多余的空白字符并删除。

```vim
highlight BadWhitespace ctermbg=red guibg=darkred
au BufRead,BufNewFile *.py,*.pyw,*.c,*.h,*.sh match BadWhitespace /\s\+$/
```

### 自动补全

使用支持vim8的[completor.vim](https://github.com/maralla/completor.vim)插件来补全代码。

安装[jedi](https://github.com/davidhalter/jedi)实现Python补全。

```cmd
pip install jedi
```

安装并配置插件。

```vim
Plugin 'maralla/completor.vim'
let g:completor_python_binary = '/path/to/python/with/jedi/installed'
```

### 语法检查

安装[syntastic](https://github.com/vim-syntastic/syntastic)插件，每次保存文件时Vim都会检查代码的语法。

```vim
Plugin 'vim-syntastic/syntastic'
```

添加[flake8](https://github.com/nvie/vim-flake8)代码风格检查。

```cmd
pip install flake8
```

安装并配置插件，保存文件时自动检查或按F7键进行检查。

```vim
Plugin 'nvie/vim-flake8'
autocmd BufWritePost *.py call Flake8()
```

### 配色方案

GUI模式可以用[solarized](https://github.com/altercation/vim-colors-solarized)方案，终端模式可以用[Zenburn](https://github.com/jnurmine/Zenburn)方案。

```vim
Plugin 'altercation/vim-colors-solarized'
Plugin 'jnurmine/Zenburn'

if has('gui_running')
  set background=dark
  colorscheme solarized
else
  colorscheme zenburn
endif
```

### 文件浏览

给vim添加一个树形目录，安装[NERDTree](https://github.com/scrooloose/nerdtree)插件，设置开关树形目录的快捷键并忽略.pyc文件。

```vim
Plugin 'scrooloose/nerdtree'
map <C-n> :NERDTreeToggle<CR>
let NERDTreeIgnore=['\~$', '\.pyc$', '\.swp$']
```

### 超级搜索/自动配对

搜索插件[ctrlp.vim](https://github.com/kien/ctrlp.vim)，自动配对符号插件[auto-pairs](https://github.com/jiangmiao/auto-pairs)。

```vim
Plugin 'kien/ctrlp.vim'
Plugin 'jiangmiao/auto-pairs'
```

### 其它

windows 10下vim8支持python3.6，按F5键保存运行python程序，在`.vimrc`文件中添加下面代码。

```vim
set pythonthreedll=python36.dll
au BufRead *.py map <buffer> <F5> :w<CR>:!python % <CR>
```
