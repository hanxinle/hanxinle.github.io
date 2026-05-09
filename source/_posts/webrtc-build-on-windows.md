---
title: WebRTC 库在 Windows 平台编译（支持 H.264）
date: 2022-09-21 15:05:11
tags:
- WebRTC
---



# 参考资料

1. [音视跳动李超老师的 WebRTC 编译博客](https://avdancedu.com/2bafd6cf/)
2. [WebRTC Support(官网)](https://webrtc.org/support/overview)

# 编译环境准备

根据官方文档：

https://webrtc.googlesource.com/src/+/main/docs/native-code/development/

<!-- more -->

Windows 平台编译 WebRTC 与编译 Chromium 软件配置和操作大多一致，可以参考编译Chromium 文档编译 WebRTC，编译Chromium 文档的网址是：

https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/windows_build_instructions.md 。

编译WebRTC需满足如下硬件要求：

1. CPU型号为 intel；
2. 内存大于等于 8G；
3. 存放源码的硬盘是 NTFS 文件系统，容量建议大于等于100G。

软件要求：

* Git

  下载地址：https://git-scm.com/

* Visual Studio 2019

  下载地址：https://visualstudio.microsoft.com/zh-hans/vs/ ,

  其它版本可在名为“MSDN,i tell you”网站下载，网址是 https://msdn.itellyou.cn/ 

  必要组件：

  选择 "Windows" 页面下的 .Net 桌面开发，使用 C++ 的桌面开发、通用 Windows 平台开发。

  在 单个组件 标签下，找到并勾选 用于x86和x64的Visual C++ MFC、用于x86和X64的Visual C++ ATL、Clang/C2(实验)、带有Spectre缓解措施的Visual C++ ATL(x86/x64)、带有Spectre缓解措施的Visual C++ MFC(x86/x64)。

* Windows sdk 和 Debugging Tools for Windows

  Windows10 编译 WebRTC 所需的 Windows SDK 版本最低要求是10.0.19041.0，如果系统没有安装满足版本要求Windows SDK。请访问  https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk/ 下载 .iso 文件，安装时选择 **Debugging Tools for Windows**。

* depot tool

depot_tool下载地址是

https://storage.googleapis.com/chrome-infra/depot_tools.zip

若文件保存路径包含中文，请将depot_tools.zip移动到**英文**路径下，例如C:\bjm_tools\，右击depot_tools.zip,选择 解压到depot_tools（英文版解压软件命令是extract files to “depot_tools\”）.

将C:\bjm_tools\depot_tools添加到系统 path 路径，然后选定该项，单击 **上移** ，将其移动到顶部，缺少这一步会导致depot_tools命令与系统原有软件命令冲突时系统不会优先选择depot_tools工具所支持的命令，产生错误。

# 编译过程

运行 x86 Native Tools Command Prompt for VS 2019 专业版，设置环境变量：

```bash
// 设置depot_tools使用本地VS进行编译
set DEPOT_TOOLS_WIN_TOOLCHAIN=0
// 设置编译工具（GYP == Generate Your Projects）
set GYP_GENERATORS=ninja,msvs-ninja
// 设置VS路径
set GYP_MSVS_OVERRIDE_PATH=C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise
// 设置VS版本
set GYP_MSVS_VERSION=2019
// 指明vs2019_install路径
set vs2019_install=C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise

```

如果是 vs2017，则设置以下变量（本人没有使用）：

```bash
// 设置depot_tools使用本地VS进行编译
set DEPOT_TOOLS_WIN_TOOLCHAIN=0
// 设置编译工具（GYP == Generate Your Projects）
set GYP_GENERATORS=ninja,msvs-ninja
// 设置VS路径
set GYP_MSVS_OVERRIDE_PATH=C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional
// 设置VS版本
set GYP_MSVS_VERSION=2017
// 指明vs2017_install路径
set vs2017_install=C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional
```

因为网络问题，我在第一次启动 cmd 的时候设置了 Git 代理

```bash
git config --global http.proxy "127.0.0.1:1080"
git config --global https.proxy "127.0.0.1:1080"
```

后来重启 cmd 又设置了系统代理（榨干ssr 最有一点流量）：

```bash
set http_proxy=127.0.0.1:1080
set https_proxy=127.0.0.1:1080
```

设置后执行:

```bash
gclient
```

该命令会安装其它所有编译 WebRTC源码所需要的、针对Windows平台特定的工具，例如 msysgit 和Python。

执行 gclient 命令后，输入命令:

```bash
where python
```

确保 python.bat 出现在任何 python.exe 前再继续操作.

设置 git ，然后下载源码:

```bash
git config --global user.name "xxxx"
git config --global user.email "xxxxx@yy.com"
git config --global core.autocrlf false
git config --global core.filemode false
git config --global branch.autosetuprebase always
```

下载源码过程是：

```bash
// 创建名为 webrtc-checkout 的文件夹
mkdir webrtc-checkout
// 进入 webrtc-checkout
cd webrtc-checkout
// 下载源码
fetch --nohooks webrtc
// 同步为最新版源码
gclient sync

# 进入 src 
cd src
```

李超老师的博客提到如下编译流程：

首先，生成目录和编译库

```bash
$ cd src
$ gn gen out/Default
$ ninja -C out/Default
```

接着，再生成 vs 工程，用于调试学习

```bash
$ gn gen --ide=vs out\Default
```

这个流程是我 2023 年 6 月份使用的，对 2021-05 和 2021-08 两个版本的源码都能编译。

在 2021 年，Default 是编译不过的。现在想当时的原因可能如下：

1. 编译环境变化，我现在把要设置的环境变量都写入到了系统环境变量中；
2. 2021 生成的待编译文件有 --ide=vs 选项，对其有干扰；
3. 现在使用了新的 Windows SDK 或者 VS2022 的关系；
4. depot_tool 的更新；
5. 命令行的更换，我使用的是 develop prompt command for vs2019;



2021 使用的生成 x86 debug with H.264 的 libwebrtc.lib 待编译文件命令是：

```bash
gn gen out/x86/vs2019ffmpeg_nolibcxx_nolld_notests --ide=vs2019 --args="target_cpu=\"x86\" is_debug = true  treat_warnings_as_errors = false
use_custom_libcxx = false use_lld = false  rtc_use_h264 = true proprietary_codecs = true is_component_ffmpeg = true enable_iterator_debugging = true ffmpeg_branding = \"Chrome\" rtc_include_tests = false"
```

| 参数                             | 含义                                           | 备注                               |
| -------------------------------- | ---------------------------------------------- | ---------------------------------- |
| out/x86/dh264ffmpeg              | 工程存放路径                                   |                                    |
| --ide=vs2017                     | 编译工程所用 IDE 工具                          |                                    |
| target_cpu=\"x86\"               | webrtc.lib 是32位                              |                                    |
| target_winuwp_family=\"desktop\" | webrtc.lib用于Windows桌面程序                  | 后续生成工程不再指定这个参数的值   |
| is_debug=true                    | 生成的静态库用于debug模式                      | 默认debug模式，查看方法见表后正文  |
| treat_warnings_as_errors=false   | 设置编译过程不将警告当作错误，避免中断编译过程 | 如果忘记关闭需要更改源码，详见后文 |
| rtc_use_h264=true                | 支持H.264编码                                  |                                    |
| proprietary_codecs=true          | 支持所有编解码方案开关开启                     |                                    |
| is_component_ffmpeg=true         | 编译ffmpeg模块用于支持H.264解码                |                                    |
| ffmpeg_branding=\"Chrome\"       | 设置编译的ffmpeg多媒体播放设置适配Chrome系统   |                                    |

上述命令耗时4336ms，Windows平台 H.264 解码用ffmpeg，ffmpeg解码需要用到Clang，is_clang选项默认为true，如果不用Clang，将不支持H.264解码。具体信息请访问 https://bugs.chromium.org/p/webrtc/issues/detail?id=9213 。4.5 节会对详细说明这个网址涉及的内容。

补充一个创建 WebRTC工程后查看配置参数的命令，即在命令行中输入

```bash
gn args out/x86/vs2019ffmpeg_nolibcxx_nolld_notests –list
```

可以查看生成该工程可设置的所有参数，以及每个参数的当前值、默认值、该值的含义等信息。

执行

```bash
gn args out/x86/vs2019ffmpeg_nolibcxx_nolld_notests –list=<参数名>
```

可以查看特定参数的值。

编译 WebRTC 工程

```
ninja -C out/x86/vs2019ffmpeg_nolibcxx_nolld_notests
```

测试编译：

进入到生成目录 *out/x86/vs2019ffmpeg_nolibcxx_nolld_notests*，运行命令保存log并查看结果.

```
vidoe_engine_tests.exe > video_engine_tests_log.txt
```



# 创建 peerconnection 工程测试编译的库

启动vs2019，打开资源管理器进入WebRTC 源码路径，在 /examples/server 下拷贝 data_socket.cc、data_socket.h、main.c、peer_channel.cc、peer_channel.h、utils.cc、utils.h 到 bjm_peerconnetcion_server 路径下，并且导入该工程；pc_client 示例程序所在路径是 src/examples/client，拷贝conductor.cc、conductor.h、defaults.cc、defaults.h、flag_defs.h、main.cc、main_wnd.cc、main_wnd.h、peer_connection_client.cc、peer_connection_server.h 到 pc_client 所在路径并且导入工程。

**对 server 配置 **include 路径和库路径，在“链接器-输入”中添加：

```
ws2_32.lib
winmm.lib
webrtc.lib
my_webrtc.lib    
```

 

PS: my_webrtc.lib 是自制.

**对 client ，配置**：

```
ws2_32.lib
winmm.lib
strmiids.lib
amstrmid.lib
dmoguids.lib
wmcodecdspuuid.lib
crypt32.lib
iphlpapi.lib
secur32.lib
rtc_json.lib
msdmo.lib
webrtc.lib
ffmpeg.dll.lib
my_webrtc.lib
```

PS: my_webrtc.lib 是自制.

编译结束后，先启动 pc_server，再启动两个 pc_client ,单击 ”connect" 按钮，计算机上如果有两个摄像头，就可以互相看到对方。



PS: 我还保留了几个脚本工具用于测试环节提取 lib、头文件、将 obj 文件制作成为 lib。



# Q&A

1、如果设置了 is_clang=false，会出现什么问题及解决方法是什么？

答：几个文件如 /modules/audio_processing/agc2/rnn_vad/features_extraction.cc

代码报错，提示不能对数据 窄化(narrowing) 操作。需要对该行报错的变量使用 static_cast<窄化后的类型> ，参考 https://en.cppreference.com/w/cpp/language/static_cast 。

另有 TaskQueue 模块个别文件提示对象访问权限问题，根据报错信息打开源码，定位到报错的行，将 protected 改为 public。

最严重的问题是无法编译 ffmpeg，因为VS对C11支持不完全，提示找不到头文件，无法通过拷贝相应头文件到VS的include 目录解决这个问题。

2、如果不设置 treat_warnings_as_errors=false 会出现什么情况？如何解决？

答：在执行ninja –C <工程路径> 命令后，编译过程会被频繁打断。需要定位到警告信息产生的代码文件相应的行，在行的前后加入 #pragma warning (disable:<警告编号>)。

3、因为编译工程频繁更改代码，如何能统计自己更改了哪些代码以及做了何种更改？

答：命令行环境进入 webrtc-checkout/src 路径,执行 git diff 即可看到对源码的修改记录。建议用Visual Studio Code(下文称VSC)修改代码，修改后保存代码但不要退出VSC，若需要恢复到修改前的代码，切换到文件标签，不断按 Cril+Z,即可撤销修改，保存文件。再执行 git diff 即可看到源码无修改。

4、如何查看自己下载的WebRTC版本信息。

答：执行 gclient revinfo –a 。

5、这些参数是不是一成不变的？开发人员可以不了解直接采用 webrtc.lib？

答：每个版本控制生成工程的参数都有可能发生变化，本文档 v1.1版撰写时 v1.0版的一些参数已经不再被使用，由其它参数覆盖、替代或干脆移除某参数涉及的模块。第二个问题的答案为是，开发人员不需要了解具体参数的含义，这也是本文档的意义——节约时间。

6、H.264 编解码支持所依赖的 ffmpeg 编译及测试方案执行

| WebRTC 生成工程参数                                   | 编译器      | C++ 标准库版本        | 链接器        | 备注               |
| ----------------------------------------------------- | ----------- | --------------------- | ------------- | ------------------ |
| is_clang=false                                        | MSVC cl.exe | MSVC lib              | MSVC link.exe |                    |
| is_clang=false  use_custom_libcxx=true  use_lld=true  | Clang       | Google custom  libcxx | lld           | 默认参数           |
| is_clang=true  use_custom_libcxx=false                | Clang       | MSVC lib              | lld           |                    |
| is_clang=true  use_custom_libcxx=false  use_lld=flase | Clang       | MSVC lib              | MSVC link.exe | 本文档使用参数设置 |

此处需要特别说明使用 H.264 编解码支持的x86 debug 版webrtc.lib 的项目属性配置方案。WebRTC 若要支持 H.264 解码，需要添加 ffmpeg 模块。编译此模块在Windows 系统需要Clang 编译器支持。因此在生成 webrtc.lib 工程时，相关参数要与上表的最后一列一致。

7、Windows 平台对 H.264 支持情况

2021年编译的时候我记录了如下信息，近期没有重新编译，进展情况自行查官方信息吧：

> WebRTC官方宣布 Windows 系统使用vs原生工具集则不支持 H.264 编解码，截至本文档撰写完毕这个 bug依旧没有解决。这是网址：
>
> [https://bugs.chromium.org/p/webrtc/issues/detail?id=9213#c13](https://bugs.chromium.org/p/webrtc/issues/detail?id=9213%23c13)
