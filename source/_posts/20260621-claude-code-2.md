---
title: Claude Code 使用备注
date: 2026-06-21 20:30:12
tags: AI Agent
---

记录一下 Claude Code 里一个状态栏插件的安装和使用。
<!-- more -->

## 安装 claude-hud

在 Claude Code 里依次执行以下命令：

```
/plugin marketplace add jarrodwatts/claude-hud
/plugin install claude-hud
/reload-plugins
```

安装完成后，运行 `/claude-hud:setup` 进行初始化配置。

## 这个插件的作用

`claude-hud` 是一个 Claude Code 的状态栏（statusline / HUD）插件，作用是在 Claude Code 界面底部显示当前状态信息，例如上下文占用、模型信息、任务进度等。它能让使用 Agent 时对当前会话的资源消耗和运行状态有更直观的感知，避免在长时间任务里对进度一无所知。

后续如果想调整显示内容，可以用 `/claude-hud:configure` 进行配置。
