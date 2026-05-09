---
title: hexo搭建环境完毕后写博客的步骤
date: 2020-10-27 23:55:57
tags: 
- 工具

mathjax:  true
---


# 一 发布博客的操作流程

1 创建新博文
``` bash
$ hexo new "博文标题"
```
<!-- more -->
编辑博客，保存。

参考: [Writing](https://hexo.io/docs/writing.html)

2 启动本地服务器地址查看博客
``` bash
$ hexo server
```

参考: [Server](https://hexo.io/docs/server.html)

3 推送到 Github(hanxinle.github.io) 
``` bash
$  hexo g && hexo d
```

PS: 下面这个命令是教程介绍的，亲测会删掉自己添加的封面等文件，慎用。
``` bash
$  hexo clean && hexo g && hexo d
```

参考: [Generating](https://hexo.io/docs/generating.html)

参考: [Deployment](https://hexo.io/docs/one-command-deployment.html)

* GitHub页面更新

更新有延迟，需要等大约1分钟，刷新博客首页可以看到新部署。

# 二 在博文中显示公式的方法

1 在需要插入公式的博客中添加设置条目mathjax并设置值为true:
```
---
title: A Title
date: 2020-02-08 10:39:55
tags:
- tag1
- tag2
categories:
- parent
- child

mathjax: true
---
```

2 主题next添加公式支持的方法【该博客采用的方案】
修改主题所在的_config.yml 文件内容如下（搜索math后修改）
```
math:
  # Default (true) will load mathjax / katex script on demand.
  # That is it only render those page which has `mathjax: true` in Front-matter.
  # If you set it to false, it will load mathjax / katex srcipt EVERY PAGE.
  enable: true
  per_page: true

  # hexo-renderer-pandoc (or hexo-renderer-kramed) required for full MathJax support.
  mathjax:
    enable: true
    # See: https://mhchem.github.io/MathJax-mhchem/
    mhchem: true
```

博客的根目录的 _config.yml 文件不需要修改。

测试公式：   $ f(x) = a+b $

3 主题butterfly的步骤有3个

* 安装插件　　
```
npm install hexo-math --save
```

* 在站点配置文件 _config.yml 中添加：　　
```
math:
  engine: 'mathjax' # or 'katex'
  mathjax:
    # src: custom_mathjax_source
    config:
      # MathJax config
```
* 在 next 主题配置文件中 themes/butterfly/_config.yml 中将 mathJax 设为 true:

```
# MathJax Support
mathjax:
  enable: true
  per_page: false
  cdn: //cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML
```
# 三 图片显示的方法

在public路径下找到和博客文件同名的文件夹，将图像文件放入其中，在博客中直接引用即可，例如下文
```
![](1.jpg)
```
显示如下：
![](1.jpg)



# 四 删掉已经发布的博客的方法

参考资料：夏普通.2020-10-13.[用Github Pages+Hexo搭建博客之(七)如何删除一篇已经发布的文章 #成功解决：同时删除掉.deploy_git文件夹](https://blog.csdn.net/qq_34243930/article/details/109046120#:~:text=%E5%85%B7%E4%BD%93%E6%AD%A5%E9%AA%A4%E5%A6%82%E4%B8%8B%201%20%E7%AC%AC%E4%B8%80%E6%AD%A5%EF%BC%8C%E5%8E%BB%E6%96%87%E4%BB%B6%E5%A4%B9%20source%2F_posts%20%E4%B8%8B%E5%88%A0%E9%99%A4%E4%BD%A0%E6%83%B3%E8%A6%81%E5%88%A0%E9%99%A4%E7%9A%84%E6%96%87%E7%AB%A0%202%20%E7%AC%AC%E4%BA%8C%E6%AD%A5%EF%BC%8C%E5%88%A0%E9%99%A4%20.deploy_git,%E5%90%8E%EF%BC%8C%E5%86%8D%E6%89%A7%E8%A1%8C%20hexo%20g%20%EF%BC%8C%20hexo%20g%20%E5%8D%B3%E5%8F%AF%E3%80%82%20%E5%8F%91%E7%8E%B0%E6%96%87%E7%AB%A0%E5%88%A0%E9%99%A4%E6%88%90%E5%8A%9F%E2%9C%94)

步骤如下：

1. 删掉 `source/_posts` 下的 .md 博客文件 ;
2. 删除 `.deploy_git` 文件夹；
3. 执行 `hexo clean`，再执行 `hexo g && hexo d`。
