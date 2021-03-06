# 安装 Ubuntu 后的配置

[返回首页](../../index.md)  

>不要花费过多时间用于折腾开发环境

# 1 lisp相关
## 1.1 emacs

外观上，偏爱黑色风格,在~/.emacs中黏贴以下内容
```
(set-background-color "black")              ;;  使用黑色背景
(set-foreground-color "white")              ;;  使用白色前景
(set-face-foreground 'region "green")       ;;  区域前景颜色设为绿色
(set-face-background 'region "blue")        ;;  区域背景色设为蓝色
(tool-bar-mode -1)                          ;;  这个关闭工具栏
;;(menu-bar-mode -1)                        ;;  这个关闭菜单栏

```
关于找不到 .emacs 文件的问题解决方法，在 Options 菜单中更改任意设置，保存该设置，就会生成 .emacs 文件，在 windows 系统中，该文件可能位于用户目录或者用户目录下子目录 \AppData\Roaming 中，可在资源管理器搜索其位置。  
自动补全，采用 auto-complete（[GitHub链接](https://github.com/auto-complete/auto-complete)），在 Ubuntu 中，解压文件到本地，得到文件夹 auto-complete-master，在 .emacs.d 文件夹中 plugins 中新建 文件夹并命名为 auto-complete，启动 emacs，执行 M-X | load-file | mydir/auto-complete-master/etc/install.el，选择安装目录是 .emacs.d/plugins/auto-complete，安装成功后根据提示 在 .emacs 中添加如下代码

```
(add-to-list 'load-path "~/.emacs.d/plugins/auto-complete")
(require 'auto-complete-config)
(ac-config-default)
```
启动 emacs，提示缺少 popup.el 文件，可以在 GitHub 上搜索该文件，或者运行
```
apt-get install elpa-popup
```
该文件的地址在[这里](https://github.com/auto-complete/popup-el),我会[备份一份](https://github.com/hanxinle/edit-tools-setting/blob/master/popup.el)。

PS：另一种不常用的方法是首先运行 ```apt-get install auto-complete-el``` ，接着在 emacs 中通过 M-X load-file 分别加载 auto-complete.el，auto-complete-config.el ，最后在 .emacs 中添加
```
(add-to-list 'load-path "/home/hanxinle/emacs/auto-complete")
(require 'auto-complete-config)
(add-to-list 'ac-dictionary-directories "/home/hanxinle/emacs/auto-complete/ac-dict")
(ac-config-default)
```
该方法执行时每个步骤都要 M-X auto-complete ,查看是否有 enable 显示。

---
ecb 会将 emacs 扩展为可以查看源码的类似 IDE 的功能，个人当前更倾向使用 Clion 等IDE。ecb 通过如下命令安装：

```
apt-get install ecb
```
启动 ecb 的命令是 M-X ecb 。

---
## 1.2 Slime

如果是初步体验这个开发环境，使用如下命令安装：
```
apt-get install slime
```
如果是 common lisp 的进阶用户，建议通过 [quicklisp](https://www.quicklisp.org/beta/) 管理开发环境，包括安装 Slime 及配置相应的开发模块等。

Slime 通过 ```M-X slime``` 启动，常用命令：
```
C-c C-d h 在线帮助

C-c C-k 编译，

C-c C-l 加载文件，

C-c C-z 回到CL-USER > 

CL-USER> (exit) 退出slime
```
本人还配置了 [Evil](https://github.com/hanxinle/evil) 环境，在 emacs 中可以综合使用 vim 和 emacs 编辑方式；同时安装了 yasnippet 。 

```
(add-to-list 'load-path "~/.emacs.d/yasnippet")
(require 'yasnippet)
(yas-global-mode 1)
(put 'upcase-region 'disabled nil)

;;add in .emacs 
;;Cril-z switch emacs-vim mode
(add-to-list `load-path "~/.emacs.d/evil")
(require `evil)
(evil-mode 1)
```
## 1.3 补充

Visual Studio Code 在编辑 .md 文件可以根据需要安装常用插件，以预览、转换成 pdf、html、jpeg 等格式，同时支持 .md 中编辑公式。  
其它开发者也会根据配置将其用于远程开发，前端开发，刷 leetcode 题目等。

# 2 系统工具

受到 这篇文章《[安装 Ubuntu 后要做的事](https://blog.csdn.net/skykingf/article/details/45267517/)》启发，经典菜单和系统指示器真乃神器也。

## 2.1 经典菜单指示器
```
sudo add-apt-repository ppa:diesch/testing
sudo apt-get update
sudo apt-get install classicmenu-indicator
```
## 2.2 系统指示器SysPeek
```
sudo add-apt-repository ppa:nilarimogard/webupd8  
sudo apt-get update  
sudo apt-get install syspeek 
```

# 3 开发工具

## 3.1 安装Opencv
```
sudo apt-get install build-essential

sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev

sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev

mkdir build
cd build

cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local ..

cmake .  

make
sudo make install

sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'

sudo ldconfig

PS editon >= 3.2 not need 
cd ..
sudo cp 3rdparty/ippicv/unpack/ippicv_lnx/lib/intel64/libippicv.a /usr/local/lib/
```
Python interface support 

```
cp /usr/local/lib/python/dist-packages/cv2.so   ~/anaconda2/lib/site-packages/
```
---

卸载 Opencv
```
sudo make uninstall
cd ..

sudo rm -r build

sudo rm -r /usr/local/include/opencv2 /usr/local/include/opencv /usr/include/opencv /usr/include/opencv2 /usr/local/share/opencv /usr/local/share/OpenCV /usr/share/opencv /usr/share/OpenCV /usr/local/bin/opencv* /usr/local/lib/libopencv*

PS: option
sudo apt-get –purge remove opencv-doc opencv-data python-opencv
```
使用opencv编译程序的方法是：
```bash
g++ xxx.cpp -o a `pkg-config --cflags --libs opencv`

./a xxx.jpg

 或者

 g++ `pkg-config --cflags opencv` xxx.cpp -o a `pkg-config --libs opencv`
./a xxx.jpg

举例：

在samples/cpp/tutorial_code/photo/decolorization中，

g++ decolor.cpp -o a `pkg-config --cflags --libs opencv`

然后， ./a 1.jpg
```
![complie_and_run_opencv.png](./img/complie_and_run_opencv.png)
## 3.2 通过 JavaScript 给 adobe reader 添加书签管理功能

在 Adobe reader 基础上采用[这本书](
https://www.amazon.com/exec/obidos/ASIN/0596006551/ref=nosim/pdftk-20)作者介绍的插件，谷歌即可找到。

购买 Adobe Acrobat 更加方便。

[返回首页](../../index.md)