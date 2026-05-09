---
title: GitHub 使用
date: 2019-09-29 01:39:23
tags:
- 工具
- Git
---

<!-- more -->

# 最新：更新一些 GitHub 使用异常的故障排查

1. 问题1： Connection reset by 20.205.243.166 port 22

​	解决方法：C://User/<usr-name> 中编辑 /.ssh/config 给文件加上以下内容后再，重启git bash。

```bash
Host github.com
    User git
    ProxyCommand connect -H 127.0.0.1:7890 %h %p
```



## 1 本地 repo

* 设置姓名和邮箱

  ```git config --global user.name "xxxx"```

  ```git config --global user.email "xxxx@yyy.zzz"```
<!-- more -->
* 创建并初始化本地空仓库

  ```git init <repo_name>```

* 查看提交的状态

  ```git status```

* 暂存区中添加更改

  ```git add <file_name>```

* 记录一行提交信息

  ```git commit -m "commit info"```

* 提交记录

  ```git log```

* 忽略特定格式文件格式

  ```touch .gitignore  && vim .gitigonre```

  ```*.out *.py```

  原有创建的格式用 git rm --cached <file_name> 忽略
  常用工程的 [ignore文件](https://github.com/github/gitignore)。

* 回滚操作

  ```
  git reset --hard <commit ID>   # <commit ID> 有两种，一种很长，一种截取前一种的若干字符表示
  git push -f -u origin main     # 好坑，参考资料写 master
  ```

* 解决push冲突

  假设用户1修改了 xxx文件，并且 push 了，我这边再修改后 push 会提示拒绝，有冲突，查看冲突：

  ```git diff master origin/master```

  建议将该文件备份，然后执行以下操作(备份，先拉取再合并：

```
git pull
vim xxx  PS 对其进行更改
git add xxx
git commit "my work"
git push
```

* GitHub 创建本地仓库缓存，在本地构建 GitHub 云端方法

  (1) 在机器 a 上执行以下语句，可在该机建立了一个暂存区。

​	```git init --bare /home/hanxinle/xdisk.git```

   （2）在机器 b 执行

```git clone hanxinle@xx.mm.yy.zz:/home/hanxinle/xdisk.git```

则将 repo 拉取到本地，在本地创建文件等更改后，在本地可执行 add，commit，push 操作。

   （3）其它机器执行

```git clone hanxinle@xx.mm.yy.zz:/home/hanxinle/xdisk.git```

则可以在得到与机器b一致的 repo。


---

## 2 远程 repo

* 查看分支

  ```git branch```

* 创建分支

  ```git branch <branch_name```

* 切换到分支

  ```git checkout <branch_name>```

* 创建并且切换

  ```git checkout -b <branch_name>```

* 分支合并(位于 master )

  ```git merge <sub_branch_name>```

  同步后可以删除这个子分支

  ```git branch -d <sub_branch_name>```

* 分支同步

  ```git push origin <loacl_branch_name>```   （将本地分支同步到服务器）

* 分支删除与重命名

  ```git brach -D/-d  PS 删除 see git -h```

  远端分支重命名(创建本地分支-->同步到远端-->删除远端同名分支-->原本地分支重命名-->推送到远端)

  ```git checkout -b <local_branch>```

  ```git push origin <local_branch>```

  ```git push --delete origin <local_name>```

  ```git commit -m <local_name> <new_branch_name>```

  ```git push origin <new_branch_name>```

* tag 管理

  网页端 release 中管理，和 commit 的哈希值相关。

---

## 3 免密操作

执行命令 ```ssh-keygen```，一路回车,在 .ssh 文件夹中, id_rsa 为私有, id_rsa.pub 为公有,在 Github 中点击右上角自己的头像,选择 setting ,在 ssh 相关设置中,点击 NEW SSH KEY ,将 id_rsa.pub 的内容复制进来,此时 push 还是需要用户名密码,有两种方式可以免输入,一种是编辑.git/config 文件，将 url=https:xxxxx 更改,它的值在网页端打开 github repo 的时候,选择 clone or download 按钮,会有 use ssh 一项,点击这一项,复制 url 链接到.git/config 文件替换 url=<url_value> 这一项.

另一种方式是在 clone 命令时,不要通过 https 命令操作,直接通过上面这个 ssh 的 url 方式操作.

---

## 4 fork 后用 upstream 同步原始 repo 最新情况

* 首先 git clone 到本地

* 查看远端分支

  ```git remote -v```

* 添加upstream分支

  ```git remote add upstream git@xxxx.git```

* 将upstream版本拉到本地

  ```git fetch upstream```

* 再合并upstream master

  ```git merge upstream/master```

* 最后push到远端

  ```git push origin master```

---

## 5 其它 repo 选项

  * Issue

    bug提交

  * Projects

    项目进度管理

  * Wiki 

    项目的介绍,可以制作网页,补充其它必要的文档,背景

  * Gist 

    代码\文档的片段

