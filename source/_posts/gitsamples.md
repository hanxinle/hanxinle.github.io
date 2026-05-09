---
title: Git 案例
date: 2023-07-22 11:29:54
tags:
- 工具
- Git
---

本文记录记录一些实际操作的案例，可以结合 [GitHub 基本操作教程](https://hanxinle.github.io/2019/09/29/github-note/) 在工作中熟悉 Git 的使用。

<!-- more -->

# 1 提交前处理冲突

场景，usr1 和 usr2 协同开发，二人修改了同样的文件，usr1 已经提交，usr2 的操作过程是：

```
git add .								# 保存更改到本地
git commit -m "<填入修改描述>"			# 提交信息，允许 git commit -am <commit-id> 追加新的 git add
# git push will rejected. 提示因为存在冲突，需要先 git pull 合并冲突

git pull								#允许 git pull 操作

接着去处理冲突文件，比如 Readme.md
<<<<< upstream

usr1 的内容
=============

usr2 的内容，也就是我的内容
>>>>> <commit-id>
```

编辑大小于号之间的内容并且保存，将 ```<<<<```、```=====```、```>>>``` 这种符号删掉，最终保存的内容就是处理的冲突保存到结果，这个时候可以编译测试。接着：

```
git add Readme.md						# 添加冲突文件
git rebase --continue					# rebase 看下参考资料
log:
	README.md: needs merge
	You must edit all merge conflicts and 											then mark them as resolved using git add
git push								
```

参考资料：

1.[见”解决冲突CONFLICT“部分](https://juejin.cn/post/6969101234338791432)

# 2 更新已有 .gitignore

更新已经在 repo 中的 .gitignore 文件，比如添加如下内容

```
bin/
bin/*
```

保存文件，之前追踪的 bin 文件夹及其内部内容，在本地立刻就不会被追踪，这个通过 git status 即可发现。即，允许不提交 .gitignore 的方式使得本地追踪生效。
