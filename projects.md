---
layout: page
title: 项目
description: 代表性项目与开源贡献
image: assets/images/projects.png
show_tile: true
nav-menu: true
---

# 项目

以下是我工作与学习中具有代表性的项目展示，更多项目请访问我的 [GitHub](https://github.com/hanxinle)。

---

## 工作项目

### 1. AI数字人编辑工具 (Windows)

- **简介**: 类似剪映的音视频剪辑软件，用于制作数字人课程。使用Qt、FFmpeg、OpenGL开发。
- **技术栈**: Qt, FFmpeg, OpenGL, C++
- **主要工作**:
  - 使用OpenGL GLSL实现遮罩算法，支持实时调整遮罩数值并查看效果
  - 通过FFmpeg编解码加速方案优化作品导出功能，4K工程导出提速明显
  - 支持拖动形式操作本地文件实现素材快速导入
  - 开发模板功能实现clip编辑自动化设置，支持自定义模板

---

### 2. AI数字人多模态交互系统 (Windows)

- **简介**: 实现AI数字人与用户互动的桌面软件，支持语音问答、人脸检测、拍照互动等功能。
- **技术栈**: Qt, libVLC, OpenCV, 科大讯飞SDK, C++
- **主要工作**:
  - 设计多进程架构，通过进程通信做协同工作
  - 开发监控模块保证系统可靠性与稳定性
  - 二次开发科大讯飞SDK实现内存自动管理
  - 实现基于音量的VAD算法模拟，提升交互体验

---

### 3. 方直金太阳教育软件

- **简介**: 具有约10余个分支的教育软件客户端，主要使用MFC和Duilib开发。
- **技术栈**: MFC, Duilib, CEF, C++
- **主要工作**:
  - 设计自动收集统计教辅资源更新功能
  - 兼容CEF新版本浏览器升级，支持跨进程传参调用
  - 集成出版社在线资源，开发点读功能
  - 设计设备码方案解决USB鉴权失效问题

---

### 4. Windows平台视频会议软件 (WebRTC)

- **简介**: 基于Google WebRTC和Qt开发的视频会议产品。
- **技术栈**: WebRTC, Qt, C++
- **主要工作**:
  - 解决WebRTC在Windows平台定制编译问题（困扰团队一年）
  - 开发客户端与信令服务器通信和WebRTC媒体协商
  - 调研SRS作为SFU的多人会议方案

---

### 5. Windows平台直播客户端 (FFmpeg)

- **简介**: 新产品技术方案预研，使用FFmpeg实现音视频处理。
- **技术栈**: FFmpeg, SDL, Qt, C++
- **主要工作**:
  - 封装解复封装、编解码、视频渲染模块
  - 实现基于Qt的FPS显示和设置功能
  - 设计独立线程类用于音视频处理模块

---

### 6. Linux平台Reactor网络通信框架

- **简介**: 无第三方依赖的C++事件驱动网络框架，可部署在嵌入式系统。
- **技术栈**: C++, Linux, Socket API
- **主要工作**:
  - 实现EventLoop、Acceptor、Channel、TcpConnection等模块
  - 从阻塞式到多线程再到I/O多路复用的演进
  - 参考muduo和Netty的设计思想

---

## 开源项目

### 7. X-Nginx

- **简介**: 基于Nginx源码在Linux实现的网络框架程序
- **链接**: [GitHub](https://github.com/hanxinle/X-Nginx)

---

### 8. X-Machine-Learning

- **简介**: 机器学习算法案例总结
- **链接**: [GitHub](https://github.com/hanxinle/X-Machine-Learning)

---

### 9. Xnlp / XChatbot

- **简介**: Linux下对话机器人项目，自然语言处理技术
- **链接**: [GitHub](https://github.com/hanxinle/Xnlp) · [GitHub](https://github.com/hanxinle/XChatbot)

---

### 10. XVideoEditor

- **简介**: Windows平台视频编辑器，使用Qt和OpenCV实现图像处理算法
- **链接**: [GitHub](https://github.com/hanxinle/XVideoEditor)

---

*更多项目请访问我的 [GitHub](https://github.com/hanxinle)*