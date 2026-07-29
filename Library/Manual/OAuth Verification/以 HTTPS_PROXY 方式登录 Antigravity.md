---
title: 以 HTTPS_PROXY 方式登录 Antigravity
created: 2026-07-26
tags:
  - Antigravity
  - Google_OAuth
---
---
# 快速开始

## 操作步骤

如果重启电脑或关闭了 PowerShell 窗口，临时环境变量会失效。每次需要 Google OAuth 认证时，重新在 PowerShell 中执行以下三行命令，再从同一窗口启动 Antigravity / Antigravity IDE：

```powershell
$env:HTTP_PROXY="http://127.0.0.1:10888"
$env:HTTPS_PROXY="http://127.0.0.1:10888"
$env:NO_PROXY="localhost,127.0.0.1"
```

在桌面或者任务管理器中找到 Antigravity / Antigravity IDE，右键菜单选择“打开文件所在的位置”，复制其 `.exe` 完整路径。

回到**刚才的同一个 PowerShell 窗口**启动 Antigravity / Antigravity IDE：


```powershell
Start-Process "C:\你的实际路径\Antigravity IDE.exe"
```

7. 在 Antigravity / Antigravity IDE 中重新进行 Google OAuth 登录。
## 注意事项

- v2RayN 必须保持运行。
- 代理端口必须与 v2RayN 当前配置一致，本机成功使用的是 `10888`。
- Antigravity 必须从设置了代理变量的同一个 PowerShell 窗口启动。
- 无须首先修改 IPv6、DNS、防火墙或全局系统代理；本次问题通过让 Antigravity 明确使用 v2RayN HTTP 代理即可解决。
- 关键是必须从设置过代理变量的 PowerShell 启动 Antigravity / Antigravity IDE，这样它创建的语言服务器等子进程才能继承代理。
- 如果仍然超时，请在 v2RayN 中开启 **TUN 模式**，彻底退出并重启 Antigravity / Antigravity IDE 后再登录。

---
# 问题描述与分析

在 Windows 上登录 Google Antigravity / Antigravity IDE 时，页面提示：

> There was an unexpected issue setting up your account

错误信息中同时出现：

```text
Post "https://oauth2.googleapis.com/token"
dial tcp ...:443
connectex: A connection attempt failed
```

这表示浏览器中的 Google 授权可能已经完成，但 Antigravity / Antigravity IDE 程序本身无法连接 Google OAuth 令牌服务器。下面是本机使用 v2RayN HTTP 代理 `127.0.0.1:10888` 后验证成功的操作流程。

# 一遍跑通的操作步骤

## 1. 确认 v2RayN 可用

1. 启动 v2RayN。
2. 选择一个可以正常访问 Google 的节点。
3. 确认 HTTP 代理地址为：

```text
http://127.0.0.1:10888
```

## 2. 完全退出 Antigravity / Antigravity IDE

关闭 Antigravity，然后打开任务管理器，确认 Antigravity 及其相关后台进程均已结束。

如果仍有 `Antigravity.exe` 等相关进程，请先结束任务，避免旧进程继续使用未配置代理的网络环境。

## 3. 设置临时代理环境变量

打开一个新的 PowerShell 窗口，依次执行：

```powershell
$env:HTTP_PROXY="http://127.0.0.1:10888"
$env:HTTPS_PROXY="http://127.0.0.1:10888"
$env:NO_PROXY="localhost,127.0.0.1"
```

这些设置只对当前 PowerShell 窗口及由它启动的程序生效，不会永久修改 Windows 的全局环境变量。

## 4. 测试 Google OAuth 连接

继续在同一个 PowerShell 窗口中执行：

```powershell
curl.exe -I -x http://127.0.0.1:10888 https://oauth2.googleapis.com/token
```

只要命令很快返回 HTTP 状态码，就说明代理已经能够连接 Google OAuth 服务器。返回 `400`、`404` 或 `405` 也不代表代理失败，因为这里仅测试网络连通性，并未提交真正的 OAuth 令牌请求。

如果命令长时间无响应或提示连接超时，请先检查：

- v2RayN 是否正在运行；
- 当前节点能否访问 Google；
- HTTP 代理端口是否仍为 `10888`。

## 5. 从同一个 PowerShell 窗口启动 Antigravity

必须从刚才设置了代理变量的 PowerShell 窗口启动 Antigravity，不能直接点击桌面或开始菜单中的快捷方式。

如果不知道程序路径：

1. 临时打开 Antigravity；
2. 在任务管理器中右键 Antigravity；
3. 选择“打开文件所在的位置”；
4. 复制 `Antigravity.exe` 的完整路径；
5. 再次完全退出 Antigravity。

随后在已设置代理变量的 PowerShell 窗口执行：

```powershell
Start-Process "C:\实际安装路径\Antigravity.exe"
```

请把示例路径替换为本机 `Antigravity.exe` 的真实完整路径。

## 6. 重新登录

在新启动的 Antigravity 中重新进行 Google 登录。此时 Antigravity 及其子进程会继承 PowerShell 中的代理环境变量，从而通过 v2RayN 访问：

```text
https://oauth2.googleapis.com/token
```

---