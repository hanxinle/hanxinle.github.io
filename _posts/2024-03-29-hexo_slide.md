---
layout: post
title: 补充 hexo 博客的使用
date: 2024-03-29 22:59:24
categories:
- 工具
tags:  工具
---

# 1 选择要保留的文件

保留_config.yml, themes/, source/, scaffolds/,package.jsona,.gitignore 这些文件夹,其它可以删除。

<!-- more -->

# 2 环境安装和恢复启用旧博客的命令

提前通过安装包安装好 Git、Node.js，然后执行：

`npm install hexo-cli -g`

`npm install`

`npm install hexo-deployer-git --save`

`hexo clean`

`hexo g`

`hexo d`

# 3 删掉某个弃用标签(tag)的方法

public\tags，删掉对应文件夹，然后执行：
`hexo g && hexo d`



# 4 图片显示的方法

在 public 路径下找到和博客文件同名的文件夹，将图像文件放入其中，在博客中直接引用即可，例如下文

```
![](1.jpg)
---

# 1 选择要保留的文件

保留_config.yml, themes/, source/, scaffolds/,package.jsona,.gitignore 这些文件夹,其它可以删除。

<!-- more -->

# 2 环境安装和恢复启用旧博客的命令

提前通过安装包安装好 Git、Node.js，然后执行：

`npm install hexo-cli -g`

`npm install`

`npm install hexo-deployer-git --save`

`hexo clean`

`hexo g`

`hexo d`

# 3 删掉某个弃用标签(tag)的方法

public\tags，删掉对应文件夹，然后执行：
`hexo g && hexo d`



# 4 图片显示的方法

在 public 路径下找到和博客文件同名的文件夹，将图像文件放入其中，在博客中直接引用即可，例如下文

```
![](1.jpg)
```
