---
title: 让 Codex 给自己中文补丁
created: 2026-07-27
tags:
  - Codex
  - Codex_Patch
---
# 问题描述

我的 Codex Desktop App 做了如下设置：

 ```plain
 File → Settings → General → Language for the app UI → "Chinese (China)"
 ```
 
 但界面语言依旧没有变，原因是在中国大陆网络环境下（墙内），中文语言包从海外CDN拉取会超时失败，导致 Codex 界面设置中文失效或退回英文，请帮我采取本地副本补丁方案来解决此问题。

# 已知条件

- Codex 桌面端是通过拉取本地 i18n JSON 资源文件来渲染中文界面的；
- Codex Desktop 已经内置`zh-CN`翻译，但`enable_i18n` `Statsig`开关默认是`false`，所以系统语言、`locale0verride=zh-CN`、 `ELECTRON_LOCALE_OVERRIDE` 都没用；
- 本地把 `get("enable_i18n"，false)` 改成`true`，翻译就能加载。但这就是我们之前尝试的`app.asar`patch路线，Windows MSIX 上会遇到签名/启动问题；
- Windows 用户反馈:`localeOverride ="zh-CN"`已保存，中文资源也在包里，但界面还是英文，日志显示`reason=statsig-disabled`；
- Windows 上 `Statsig `初始化失败会同时导致`i18n`和`Browser Use`被禁用。日志里有
`user.custom.workspace_type: Invalid input`、`reason=statsig-disabled`。


# 任务限制

- 首先排除缓存干扰‌：需清除`~/.codex/cache`和`~/.codex/webviewCache`目录，重新打开 Codex 静置1-2分钟让语言包加载完成。
- 不改官方Codex。
- 不重签MSIX，因为 Windows 的 Codex Desktop App 安装是msix中安装，这类的安装包限制高。
- 不要求用户信任证书。
- 不去碰`C:\Program Files\WindowsApps` 里的官方安装目录。
- 把官方Codex的 app 目录复制到用户目录:`%LOCALAPPDATA%\OpenAI\CodexZh\app`
- 在复制出来的 `app.asar` 里，把 `enable_i18n` 默认值从 `false` 改成 `true`.
- 给这个复制版创建桌面快捷方式`Codex Zh`。
- 启动时额外带上中文参数和独立用户数据目录，避免和官方Codex冲突。
