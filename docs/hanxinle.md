# 测试自己加MD文件

## 1 算法
把<<[剑指offer](InterviewQuestions)>>(第一版)的放上来,书中写着运行环境是VS2008,我的任务是除了学习作者的程序,算法外,将其改为更加现代的C++实现，作为额外的练习．

STL可以看侯捷的视频，以及这个repo中的stl文件夹，[CPPCON2019播放列表](https://www.youtube.com/watch?v=u_ij0YNkFUs&list=PLHTh1InhhwT6KhvViwRiTR7I5s09dLCSw)也是不错的，集合了全世界的CPP专家的经验，而且不收费，代码和PPT可以在视频简介的链接中找到。

## 2 [Unix 网络编程](unix_network_programming)

使用TCP协议发送、接收文件，客户端发送文件，服务端读取客户端发送的文件名及文件内容，（目前存在一些问题，在尝试解决）。

更新：问题解决，是client客户端文件名字发送问题，缓冲区过大，将缓冲区只设置为与文件名大小一致即可，多一个字节可能会访问越界（Ubuntu 18.04 LTS）。

发现 tcpserver.cpp设计存在一个问题：无法获取客户端正确的ip地址，解决方法是将 server、client的socket_in结构体声明为私有成员，目的是使其具有与对象一致的存在周期，解决了这个问题，参考 tcpserver_cli_ip_OK.cpp。

## 3 [STL](./stl/README.md)

## 4 [设计模式](./desing_patterns/README.md)

测试自己添加图片
![图片](../img/lambda.jpg)
