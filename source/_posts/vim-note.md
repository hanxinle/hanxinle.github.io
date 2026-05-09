---
title: Vim 教程
date: 2021-08-31 09:34:43
tags:
- 工具
---



# 个性化的配置文件
<!-- more -->
注释是一个双引号 "

```
"plugins manage
call plug#begin()
Plug 'ycm-core/YouCompleteMe'
call plug#end()

packloadall
silent! helptags ALL

noremap ; :
inoremap jk <esc>
inoremap ' ''<esc>i
inoremap " ""<esc>i
inoremap ( ()<esc>i
inoremap [ []<esc>i
inoremap { {}<esc>i

noremap <f1> <c-]>
noremap <f2> <c-t>

syntax on
filetype plugin indent on

set autoindent
set expandtab
set tabstop=4
set shiftwidth=4
set backspace=2

colorscheme murphy
```

# 版本恢复，错误处理

.swap 文件，能够防止 ssh 中断而导致编写的文件内容丢失，若当前目录下存在 .swap 文件，书上说是可以设置跳转文件的路径的。

可选操作：

`r` 恢复内容

`d` 删除内容

设置 .swap 目录，防止多个 .swap 文件污染当前路径，在 .vimrc 中输入

```
set directory=$HOME/.vim/swap//   # 最后是两个斜杠 //
```

#  查找/跳转
##  文件内跳转

| 按键                | 操作                             | 备注                                           |
| ------------------- | -------------------------------- | ---------------------------------------------- |
| `hjkl`              | 方向跳转                         | 搭配数字操作更快，如 `22j`，向下跳转 22 行     |
| `{ / }`             | 段落跳转                         |                                                |
| `( / )`             | 句子开头、结尾                   |                                                |
| `gg/G`              | 文件尾/文件头                    |                                                |
| `Ctrl +b / Ctrl +f` | 翻页                             |                                                |
| `WBE/wbe`           | 逐单词跳转                       | 区别在于大写字母操作跳转力度大                 |
| `f/F <字母>`        | 同一行前后查找、跳转             | 向前/向后 跳转到匹配的第一个字母处             |
| `t/T <字母>`        | 同一行前后查找、跳转             | 向前/向后 跳转到匹配的第一个字母的前一个字符处 |
| `d f <字母>`        | 删除所有字符，到匹配的第一个字母 |                                                |

- 其它跳转操作 

| 按键                        | 操作                              | 备注 |
| --------------------------- | --------------------------------- | ---- |
| `: /<string > `             | `查找 <string>`，`n` 下个查找结果 |      |
| `*`                         | 同一个单词跳转                    |      |
| `'. (单引号，后句号)`       | 上一次修改的行                    |      |
| `. (esc 下面的按键，后句号) | 上一次修改的点                    |      |





## 文件间跳转

首先，未启动vim 时打开多个文件：

`vim file1 file2 ...` 便可以打开所有想要打开的文件

若 vim 已经启动：

输入

```
:e file
```

可以再打开一个文件，并且此时vim里会显示出file文件的内容。

同一个窗口中切换文件

`Ctrl+6`     两文件间的切换，打开过两个文件才能切换

`:bn`          下一个文件

`:bp`          上一个文件

组合拳：

`:ls`          列出打开的文件，带编号

`:b1~n`      切换至第n个文件

# 窗口切分、跳转、移动、调整大小

`:sp`       垂直切分窗口

`:vs`       水平切分窗口

`C-w-r`   切换窗口；

`C-w-c`   关闭窗口；

`Ctrl+w+方向键`    切换到前／下／上／后一个窗格

`Ctrl+w+h/j/k/l`   同上

`Ctrl+ww`              依次向后切换到下一个窗格中

移动整个窗口内容方便查看：

`Ctrl+w，HJKL`    将当前窗口移动到屏幕的 左/底/上/右

调整窗口大小的命令：

`Ctrl+w，=`         所有窗口恢复到同样大小

`:resize +<数字>`  高度增加 N 行

`:resize -<数字>`  高度减小 N 行

`:vertical resize +<数字>`    宽度增加 

`:vertical resize -<数字>`    宽度减小

# 文本操作

`c` 修改命令，如 `cw`  输入后就进入插入模式

`o` 另起一行并进入插入模式

`x` 删除一个字符

`d` 剪切，当删除用

`y` 复制

`p` 粘贴

#  插件

##  安装 vim

```
git clone https://github.com/vim/vim.git
cd vim/src
./configure --enable-python3interp=yes --enable-cscope --enable-fontset --with-python3-config-dir=/usr/lib/python3.6/config-3.6m-x86_64-linux-gnu --with-python3-command=/usr/bin/python3.6
make
sudo make install
```

##  安装插件管理工具 vim-plug

```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.github.com/junegunn/vim-plug/master/plug.vim
```

设置 .vimrc （注意：那两个# 不是注释），把要安装的插件名字放在单引号中，就复制GitHub 用户+ 仓库名就行，以 `YouCompleteMe` 为例。

```
call plug#begin()
Plug 'ycm-core/YouCompleteMe'
call plug#end()
```

启动 vim，执行以下命令安装插件。

```
:PlugInstall 
```

## 安装 `YouCompleteMe`

对于 `YouCompleteMe`，安装一是要KeXue上网同步代码，二是要在里面执行下安装脚本 install.sh，这样在启动的时候才不会提示 ycmd 服务器没有启动的问题。

我在 Ubuntu18.08 LTS 上成功安装的，记录下过程。

升级安装gcc-8、g++-8，这个 apt 命令就搞定，还得执行个命令（[参考文献](https://blog.csdn.net/maoni99999/article/details/117378664)）

```
sudo apt-get install g++-8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 800 --slave /usr/bin/g++ g++ /usr/bin/g++-8
```

然后进入到 `YouCompleteMe` 目录，执行

```
sh install.sh        # 或者 ./install.sh  
```

我执行 `./install.py --all` 中间会报错。

## 读源码的工具 ctags

```
sudo apt-get install ctags
```

进入源码目录，执行

```
ctags -R
```

用 vim 打开文件，想查询定义就按 <f1>，想看声明就按 <f2>，这两个键在 .vimrc 中有配置。取代了 `Crtl+t`/`Crtl+]`。

# 在 Visual Studio 等 IDE 中常用的设置
```
# 输入模式下：jk 代替 esc ，减少左手找案件的操作
imap jk <esc>

# 命令模式下，分号代替冒号，减少按 shift 的操作
noremap ; :
```

# **参考资料**

[1] [w3cscchool 的教程](https://www.w3cschool.cn/vim/cjtr1pu3.html)

[2] [慕课网 PegasusWang 的高质量免费教程](https://www.imooc.com/learn/1129)

[3] [Chrome插件Vimium使用教程](https://zhuanlan.zhihu.com/p/113316942)

