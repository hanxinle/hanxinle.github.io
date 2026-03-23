---
layout: post
title: Ubuntu/Winodws 开发环境打点备忘
date: 2023-06-19 16:59:15
categories:
- 技术
- 编程语言
tags:
- 工具
---

# 1 引言

之前写过 vim 的教程，还写过 emacs 使用教程，自己整理了一份 emacs.d 文件，每次装机了就拿出来。

这一次是配置共享文件，这个不仅可以用于虚拟机，也可以用于 Windows 系统与远程开发机同步文件，并且方便在本地使用一些编码工具。

<!-- more -->

刚刚在 ip 地址是 192.168.0.3 的 vmware pro 虚拟机搭建成功了，暂时告别 vmware 自带的共享文件夹了，因为共享文件夹的权限问题，有些文件编译不成功，所以想到用 samba服务，在上家公司工作的时候，需要帮着嵌入式部门解决问题，这个服务其实也没有配置成功，真的是蛋疼，工作效率别提多低了。

# 2 Samba 安装步骤

1. 执行命令行

   ```
   sudo apt-get install samba
   ```

2. 编辑 /etc/smb.conf，跳到文件尾部，插入如下内容

   ```C++
   [smb_share] # 会在共享中显示的名字
   	comment = share
   	path = /home/hanxinle/smb_share # 绝对路径，记得去创建文件夹
   	writable = yes
   	browseable = yes
   ```

3. 创建 smba 用户，按照提示出入密码

   ```
   sudo smbpasswd -a hanxinle
   ```

4. 重启服务，实际上我是先重启后，无法使用服务才根据资料创建的 smba 用户

   ```bash
   sudo /etc/init.d/smbd restart
   sudo /etc/init.d/nmbd restart
   systemctl restart smb
   ```

5. Winodws 文件夹中输入 \\192.168.0.103，会弹出名为 smb_share 的网络资源，打开后进入，这一步骤可能要输入用户名和密码。

6. 可以下一次输入 \\\\ip 后，右击网络资源，选择映射到本地磁盘，选择一个没有使用的盘符，进入后编辑的文件会自动同步到 Ubuntu 系统.

   

# 3 Windows 按照域名访问 vmware 虚拟机

首先，VM 中的 Ubuntu 以 root 权限编辑 /etc/hosts 文件，假设虚拟机 ip 是 192.168.0.103 则添加如下内容：

```
192.168.0.103   rtc.com
```

在 Windows 10 中找到路径 *C:\Windows\System32\drivers\etc*，以管理员身份打开该路径中的 *hosts* 文件，添加如下内容：

```
192.168.0.103   rtc.com
```

Windows 10 中执行 ping 命令即可验证。

# 4 Windows 系统局域网传输文件

---

# 1 引言

之前写过 vim 的教程，还写过 emacs 使用教程，自己整理了一份 emacs.d 文件，每次装机了就拿出来。

这一次是配置共享文件，这个不仅可以用于虚拟机，也可以用于 Windows 系统与远程开发机同步文件，并且方便在本地使用一些编码工具。

<!-- more -->

刚刚在 ip 地址是 192.168.0.3 的 vmware pro 虚拟机搭建成功了，暂时告别 vmware 自带的共享文件夹了，因为共享文件夹的权限问题，有些文件编译不成功，所以想到用 samba服务，在上家公司工作的时候，需要帮着嵌入式部门解决问题，这个服务其实也没有配置成功，真的是蛋疼，工作效率别提多低了。

# 2 Samba 安装步骤

1. 执行命令行

   ```
   sudo apt-get install samba
   ```

2. 编辑 /etc/smb.conf，跳到文件尾部，插入如下内容

   ```C++
   [smb_share] # 会在共享中显示的名字
   	comment = share
   	path = /home/hanxinle/smb_share # 绝对路径，记得去创建文件夹
   	writable = yes
   	browseable = yes
   ```

3. 创建 smba 用户，按照提示出入密码

   ```
   sudo smbpasswd -a hanxinle
   ```

4. 重启服务，实际上我是先重启后，无法使用服务才根据资料创建的 smba 用户

   ```bash
   sudo /etc/init.d/smbd restart
   sudo /etc/init.d/nmbd restart
   systemctl restart smb
   ```

5. Winodws 文件夹中输入 \\192.168.0.103，会弹出名为 smb_share 的网络资源，打开后进入，这一步骤可能要输入用户名和密码。

6. 可以下一次输入 \\\\ip 后，右击网络资源，选择映射到本地磁盘，选择一个没有使用的盘符，进入后编辑的文件会自动同步到 Ubuntu 系统.

   

# 3 Windows 按照域名访问 vmware 虚拟机

首先，VM 中的 Ubuntu 以 root 权限编辑 /etc/hosts 文件，假设虚拟机 ip 是 192.168.0.103 则添加如下内容：

```
192.168.0.103   rtc.com
```

在 Windows 10 中找到路径 *C:\Windows\System32\drivers\etc*，以管理员身份打开该路径中的 *hosts* 文件，添加如下内容：

```
192.168.0.103   rtc.com
```

Windows 10 中执行 ping 命令即可验证。

# 4 Windows 系统局域网传输文件

将要分享的文件放入文件夹，然后将该文件夹分享给 Everyone，另一台 Windows 中 *WIN+R* 后执行 \\\<pc1的ip>，即可访问分享的文件夹，和配置 samba 并访问相似。 
