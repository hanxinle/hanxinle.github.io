---
layout: post
title: 开发环境问题处理
date: 2023-12-02 14:08:34
categories:
- 工具
tags:
- 工具
---

# 1 一些教训

非必要不升级系统，导致很多经典软件不能用，最后重新安装系统解决这些问题的.

<!-- more -->

# 2 樱桃键盘



| 问题                 | 按键    |
| -------------------- | ------- |
| win 键盘失效         | fn+F9   |
| F1~F3 与音量强制关联 | fn+Ctrl |

# 3 Git 状态图标异常

`WIN+R` 启动注册表，找到这一项：

```
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers\
```

删掉该项下的 `  Tortoise1Normal` 前的所有内容，任务管理器中重启 EXPLORER。



---

# 1 一些教训

非必要不升级系统，导致很多经典软件不能用，最后重新安装系统解决这些问题的.

<!-- more -->

# 2 樱桃键盘



| 问题                 | 按键    |
| -------------------- | ------- |
| win 键盘失效         | fn+F9   |
| F1~F3 与音量强制关联 | fn+Ctrl |

# 3 Git 状态图标异常

`WIN+R` 启动注册表，找到这一项：

```
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers\
```

删掉该项下的 `  Tortoise1Normal` 前的所有内容，任务管理器中重启 EXPLORER。



# 4 
