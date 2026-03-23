---
layout: post
title: SRS 多种手段推拉流
date: 2021-07-18 17:34:11
tags:
- SRS
---

这篇文章记录几种与 SRS 之间推拉流的方法。
<!-- more -->
# docker
Docker 安装在 Windows 平台使用，这里备注两个使用时遇到的坑。一个是关于镜像保存，另一个是关于使用环境。容易使用的镜像因为对端口等有修改，所以每次使用后需要 commit，本地备份镜像的命令是：
```
docker commit <image-id> <usr-defined-name>
```
<usr-defined-name> 的建议的格式如下，不遵守格式会引发问题。
```
<usr-name>/<container-name>:<tag>
```
我使用的名字是benjamin199/srs-4release:latest。
commit后的推送命令是：
```
docker push benjamin199/srs-4release:latest
```
推送时如果遇到如下报错：denied: requested access to the resource is denied.这就是因为镜像命名不规范而引发的错误，按照上述重新 commit 并且push可以解决这个问题。

docker在Windows上使用还有其它的一些坑，需要如下操作避开。
首先，按照docker的教程开启 Hyper-V、Linux子系统，后续会少麻烦。另外，要在应用商店搜索Ubuntu，我安装的是Ubuntu18（不收费）。
其次，如果启动docker提示 WSL无法启动，正确的解决方法是：按照老外的方法，卸载docker重新安装最新版，不要花费时间去重新设置docker，重新设置的方式行不通。
最后，无法加载镜像，docker无法启动，提示端口被占用，或者permission dennied, can not access 等。遇到这类情况，重启系统。如果还有这类问题，重新安装最新版docker。

docker部署后的推拉流命令分别如下：
```
ffmpeg -re -i a.flv -vcodec copy -acodec copy -f flv -y rtmp://localhost/live/livestream

ffplay -x 640 -y 480 rtmp://localhost/live/livestream
```
# obs
obs推拉流的坑在于密钥，实际上这是流的名字。使用obs的时候发现遇到一个问题，就是64位的软件会崩溃，因此操作使用32位。
# webrtc拉流
webrtc拉流可以使用的配置文件是rtc2rtmp.conf，把rtc_server中的candidata修改为云主机的外部ip，启动srs
```
cd srs/trunk
./objs/srs -c conf/rtc2rtmp.conf
```
启动srs后，本地需要推流以产生视频源，
```
ffmpeg -re -i a.flv -vcodec copy -acodec copy -f flv -y rtmp://<domain-name or ip>/live/livestream
```


# vlc
## vlc推流
菜单栏选择“媒体”-“流”，在“打开媒体”中选择“添加”添加媒体后点击“串流”，在“流输出”中设置rtsp后点击添加，端口6554不变，路径填上11，“流输出”的“激活转码”不选择，“串流所有基本流”也不选择，最后单击“流”。

## vlc拉流
“媒体”——“打开网络串流”，如果是rtsp链接，可以填写如下rtsp://@:8554/11，其中@表示localhost，有些时候不添加可能会掉坑。

# FFmpeg推拉流

获取摄像头名称：
```
ffmpeg -list_devices true -f dshow -i dummy
```
摄像头推流命令，其中-s控制的是分辨率:
```
ffmpeg -f dshow -i video="HD Webcam eMeet C960" -vcodec libx264 -acodec copy -preset:v ultrafast -tune:v zerolatency -s 320x240 -f flv rtmp://<ip or domain-name>/live/livestream4
```
文件推流命令：
```
ffmpeg -re -i a.flv -vcodec copy -acodec copy -f flv -y rtmp://localhost/live/livestream
---

这篇文章记录几种与 SRS 之间推拉流的方法。
<!-- more -->
# docker
Docker 安装在 Windows 平台使用，这里备注两个使用时遇到的坑。一个是关于镜像保存，另一个是关于使用环境。容易使用的镜像因为对端口等有修改，所以每次使用后需要 commit，本地备份镜像的命令是：
```
docker commit <image-id> <usr-defined-name>
```
<usr-defined-name> 的建议的格式如下，不遵守格式会引发问题。
```
<usr-name>/<container-name>:<tag>
```
我使用的名字是benjamin199/srs-4release:latest。
commit后的推送命令是：
```
docker push benjamin199/srs-4release:latest
```
推送时如果遇到如下报错：denied: requested access to the resource is denied.这就是因为镜像命名不规范而引发的错误，按照上述重新 commit 并且push可以解决这个问题。

docker在Windows上使用还有其它的一些坑，需要如下操作避开。
首先，按照docker的教程开启 Hyper-V、Linux子系统，后续会少麻烦。另外，要在应用商店搜索Ubuntu，我安装的是Ubuntu18（不收费）。
其次，如果启动docker提示 WSL无法启动，正确的解决方法是：按照老外的方法，卸载docker重新安装最新版，不要花费时间去重新设置docker，重新设置的方式行不通。
最后，无法加载镜像，docker无法启动，提示端口被占用，或者permission dennied, can not access 等。遇到这类情况，重启系统。如果还有这类问题，重新安装最新版docker。

docker部署后的推拉流命令分别如下：
```
ffmpeg -re -i a.flv -vcodec copy -acodec copy -f flv -y rtmp://localhost/live/livestream

ffplay -x 640 -y 480 rtmp://localhost/live/livestream
```
# obs
obs推拉流的坑在于密钥，实际上这是流的名字。使用obs的时候发现遇到一个问题，就是64位的软件会崩溃，因此操作使用32位。
# webrtc拉流
webrtc拉流可以使用的配置文件是rtc2rtmp.conf，把rtc_server中的candidata修改为云主机的外部ip，启动srs
```
cd srs/trunk
./objs/srs -c conf/rtc2rtmp.conf
```
启动srs后，本地需要推流以产生视频源，
```
ffmpeg -re -i a.flv -vcodec copy -acodec copy -f flv -y rtmp://<domain-name or ip>/live/livestream
```


# vlc
## vlc推流
菜单栏选择“媒体”-“流”，在“打开媒体”中选择“添加”添加媒体后点击“串流”，在“流输出”中设置rtsp后点击添加，端口6554不变，路径填上11，“流输出”的“激活转码”不选择，“串流所有基本流”也不选择，最后单击“流”。

## vlc拉流
“媒体”——“打开网络串流”，如果是rtsp链接，可以填写如下rtsp://@:8554/11，其中@表示localhost，有些时候不添加可能会掉坑。

# FFmpeg推拉流

获取摄像头名称：
```
ffmpeg -list_devices true -f dshow -i dummy
```
摄像头推流命令，其中-s控制的是分辨率:
```
ffmpeg -f dshow -i video="HD Webcam eMeet C960" -vcodec libx264 -acodec copy -preset:v ultrafast -tune:v zerolatency -s 320x240 -f flv rtmp://<ip or domain-name>/live/livestream4
```
文件推流命令：
```
ffmpeg -re -i a.flv -vcodec copy -acodec copy -f flv -y rtmp://localhost/live/livestream
```
