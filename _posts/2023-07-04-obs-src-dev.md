---
layout: post
title: 使用 OBS 开发零散记录
date: 2023-07-04 09:52:56
categories:
- 技术
tags:
- OBS
---

这个笔记比较乱，记录下 OBS 源码分析和使用 OBS 开发的一些做法。

<!-- more -->

# 我对源码修改的地方：

1. I:\007_webrtc_ffmpeg_opencv_obs\obs_2019\obs-studio\plugins\win-capture\window-capture.c

line 407, 最小化处理，原代码注释掉了。

2. I:\007_webrtc_ffmpeg_opencv_obs\obs_2019\obs-studio\libobs\obs-windows.c

将三处 data、plugin 的加载路径由上层改为本地。

# 分析源码

![image-20230701181056276](image-20230701181056276.png)

代码的四大块，其中 frontend 是页面。

![image-20230701181903678](image-20230701181903678.png)

主界面：OBSBasic.ui，

F10，调试模式进入到 main 函数，解决方案里面搜 main( 不管用。

I:\007_webrtc_ffmpeg_opencv_obs\obs_2019\obs-studio\UI\obs-app.cpp

line: 2126 中，找到初始化语句 *program.OBSInit()*，在这个函数中，有条语句是：

*qRegisterMetaType<VoidFunc>()*，使用的信号不是基本信号的时候，要记得对信号进行注册，不然信号发不过去。

继续跟踪函数，会看到追踪至 StartupOBS，内部有 obs_stratup，这个函数查看声明，前缀带有 EXPORT，说明是有库将其导出的，因此在后续的开发中，我们可以只调用这个函数开发就可以了。



初始化中需要初始化音视频，创建基本界面，还会检查更新等，对我们来说关心的就是初始化音频和视频。

obs 中加载插件的方式是 Loadlibray 和 GetProcAddress ，即加载库然后获取函数的名字再使用，程序销毁的时候做 Free 操作。还是再 OBSInit 中查看这些信息。

obs_load_modules() ;

音频设置 

ResetAudio

![image-20230701224233845](image-20230701224233845.png)

![image-20230701224142137](image-20230701224142137.png)

![image-20230701225306546](image-20230701225306546.png)

obs_reset_audio () 后续会用到这个库，是 libobs 导出的.

obs_init_audio () 

视频设置

ResetVideo

AttemptToResetVideo

obs_reset_video

obs_init_video

obs 根对象

struct obs_core * obs = NULL; 整个工程的功能都依赖这个对象，比如音视频帧的获取等等。

创建图像线程 obs_graphices_thread 预览功能。



os_semaphore 是信号量相关，再video-io.c 中，点击录制会进入到其中执行有关逻辑。

ResetOutputs（） 设置输出对象---obs_output_create--录制相关，mux 复用器, 属于 ffmpeg 的复用。

点击录制就会还会计进入 obs-ffmpeg-mux.c 中 ffmpeg_mux_start (void * data) 开始的

# 从录制开始

recordButton 以及槽函数 on_recordButton_clicked

obs_output_start、obs_output_stop 是以后我们会学习使用的两种不同的流程。

声音的采集需要有混音操作，可能比画面录制更复杂。

窗口采集用 bitblt 只能采集有窗口句柄的窗口，无句柄的用 wgc,例如谷歌浏览器或者 vscode ，继续使用 bitblt 会出现黑屏。

显示器采集源码在 duplicator-monitor-capture.c 

bitblt x264 采集源码的详细分析

点击开始录制按钮后发生了什么？



什么时候开始编码？





视频帧是怎么存储？

环形队列 circleBuffer 的方式.

4个线程

video_thread ：录制编码

obs_graphics_thread 预览、采集

audio_thread 音频输出

CaptureThread 音频采集线程

raw_frame 是信号量，

os_sem_post os_sem_wait

大项目记得看堆栈

Load(savePath) 进入 win-wasapi 是音频采集相关的项目，

osb-source.c 中有处理多路音频的数据，要比视频处理复杂。



# 开始移植工程

学到了一个技巧，

点击要查看的函数下断点，当执行到这里的时候，就可以看到程序填入的实际值，那么自己移植程序的时候，就可以将值直接填入然后继续移植工作，不需要再去代码中查找。



带界面的窗口出问题记得使用 exit(0);



---

这个笔记比较乱，记录下 OBS 源码分析和使用 OBS 开发的一些做法。

<!-- more -->

# 我对源码修改的地方：

1. I:\007_webrtc_ffmpeg_opencv_obs\obs_2019\obs-studio\plugins\win-capture\window-capture.c

line 407, 最小化处理，原代码注释掉了。

2. I:\007_webrtc_ffmpeg_opencv_obs\obs_2019\obs-studio\libobs\obs-windows.c

将三处 data、plugin 的加载路径由上层改为本地。

# 分析源码

![image-20230701181056276](image-20230701181056276.png)

代码的四大块，其中 frontend 是页面。

![image-20230701181903678](image-20230701181903678.png)

主界面：OBSBasic.ui，

F10，调试模式进入到 main 函数，解决方案里面搜 main( 不管用。

I:\007_webrtc_ffmpeg_opencv_obs\obs_2019\obs-studio\UI\obs-app.cpp

line: 2126 中，找到初始化语句 *program.OBSInit()*，在这个函数中，有条语句是：

*qRegisterMetaType<VoidFunc>()*，使用的信号不是基本信号的时候，要记得对信号进行注册，不然信号发不过去。

继续跟踪函数，会看到追踪至 StartupOBS，内部有 obs_stratup，这个函数查看声明，前缀带有 EXPORT，说明是有库将其导出的，因此在后续的开发中，我们可以只调用这个函数开发就可以了。



初始化中需要初始化音视频，创建基本界面，还会检查更新等，对我们来说关心的就是初始化音频和视频。

obs 中加载插件的方式是 Loadlibray 和 GetProcAddress ，即加载库然后获取函数的名字再使用，程序销毁的时候做 Free 操作。还是再 OBSInit 中查看这些信息。

obs_load_modules() ;

音频设置 

ResetAudio

![image-20230701224233845](image-20230701224233845.png)

![image-20230701224142137](image-20230701224142137.png)

![image-20230701225306546](image-20230701225306546.png)

obs_reset_audio () 后续会用到这个库，是 libobs 导出的.

obs_init_audio () 

视频设置

ResetVideo

AttemptToResetVideo

obs_reset_video

obs_init_video

obs 根对象

struct obs_core * obs = NULL; 整个工程的功能都依赖这个对象，比如音视频帧的获取等等。

创建图像线程 obs_graphices_thread 预览功能。



os_semaphore 是信号量相关，再video-io.c 中，点击录制会进入到其中执行有关逻辑。

ResetOutputs（） 设置输出对象---obs_output_create--录制相关，mux 复用器, 属于 ffmpeg 的复用。

点击录制就会还会计进入 obs-ffmpeg-mux.c 中 ffmpeg_mux_start (void * data) 开始的

# 从录制开始

recordButton 以及槽函数 on_recordButton_clicked

obs_output_start、obs_output_stop 是以后我们会学习使用的两种不同的流程。

声音的采集需要有混音操作，可能比画面录制更复杂。

窗口采集用 bitblt 只能采集有窗口句柄的窗口，无句柄的用 wgc,例如谷歌浏览器或者 vscode ，继续使用 bitblt 会出现黑屏。

显示器采集源码在 duplicator-monitor-capture.c 

bitblt x264 采集源码的详细分析

点击开始录制按钮后发生了什么？



什么时候开始编码？





视频帧是怎么存储？

环形队列 circleBuffer 的方式.

4个线程

video_thread ：录制编码

obs_graphics_thread 预览、采集

audio_thread 音频输出

CaptureThread 音频采集线程

raw_frame 是信号量，

os_sem_post os_sem_wait

大项目记得看堆栈

Load(savePath) 进入 win-wasapi 是音频采集相关的项目，

osb-source.c 中有处理多路音频的数据，要比视频处理复杂。



# 开始移植工程

学到了一个技巧，

点击要查看的函数下断点，当执行到这里的时候，就可以看到程序填入的实际值，那么自己移植程序的时候，就可以将值直接填入然后继续移植工作，不需要再去代码中查找。



带界面的窗口出问题记得使用 exit(0);



obs 要先创建场景再创建源，然后才能使用。
