---
title: CLIProxyAPI-Antigravity-代理配置指南
created: 2026-07-25
tags:
  - CLIProxyAPI
  - Antigravity
  - CLIProxyAPI_Antigravity
---
---
# CLIProxyAPI + Antigravity + VS Code：直通成功配置指南

本指南适用于 Windows、CLIProxyAPI 由 `Antigravity for Copilot`这个 Visual Studio Code 扩展自动启动、且本机 HTTP 代理为 `http://127.0.0.1:10888` 的场景。

## 目标

让 CLIProxyAPI 能通过本地代理完成 Antigravity 的 Google OAuth 登录，并让 VS Code/Copilot 经由本地服务正常使用模型。

## 1. 确认本地 HTTP 代理已启动

确认代理软件正在运行，且 HTTP（或 Mixed）监听端口为 `10888`。

用 PowerShell 验证代理能访问 Google：

```powershell
curl.exe -x http://127.0.0.1:10888 -I https://oauth2.googleapis.com
```

只要命令返回任意 HTTP 状态码（例如 `404 Not Found`），即代表代理链路可用。`404` 在这里是正常的：该测试向根路径发出 HEAD 请求，根路径本身没有对应资源；关键是不能出现连接超时或拒绝连接。

> 不要用 `Test-NetConnection oauth2.googleapis.com -Port 443` 作为代理配置后的最终判定。它测试的是**直连** Google；直连被拦截时该命令仍会失败，但通过 HTTP 代理的请求可以成功。

## 2. 为 CLIProxyAPI 配置持久代理（最关键）

打开配置文件：

```text
C:\Users\Administrator\CLIProxyAPI\config.yaml
```

在 YAML 的**顶层**加入：

```yaml
proxy-url: "http://127.0.0.1:10888"
```

一个精简可用的示例：

```yaml
port: 8317
host: "127.0.0.1"
auth-dir: "C:\\Users\\Administrator\\.cli-proxy-api"
proxy-url: "http://127.0.0.1:10888"

providers:
  antigravity:
    enabled: true
```

注意事项：

- `proxy-url` 必须与 `port`、`host`、`providers` 同级，**不要**缩进到 `providers:` 或 `antigravity:` 下。
- 上面地址是 HTTP 代理地址；若实际使用 SOCKS5 端口，应改成对应的 `socks5://127.0.0.1:端口`。
- 这一步比在 PowerShell 中设置 `HTTP_PROXY` 更可靠，因为 CLIProxyAPI 是由 VS Code 扩展启动的，通常不会继承另一个 PowerShell 窗口的临时环境变量。

## 3. 为 VS Code 配置同一代理

在 VS Code 中按 `Ctrl+Shift+P`，执行：

```text
Preferences: Open User Settings (JSON)
```

在用户级 `settings.json` 加入：

```json
{
  "http.proxy": "http://127.0.0.1:10888",
  "http.proxySupport": "override"
}
```

如果文件已有其他配置，保留原有内容，并确保 JSON 条目之间有逗号。保存后按 `Ctrl+Shift+P`，执行：

```text
Developer: Reload Window
```

不建议为此问题关闭 TLS 校验（例如设置 `http.proxyStrictSSL: false`）。只有在明确使用且信任带自签名证书的企业代理时，才应单独处理证书问题。

## 4. 重启并完成 Antigravity 登录

1. 停止当前由扩展启动的 CLIProxyAPI 服务。
2. 确认 `config.yaml` 已保存了 `proxy-url`。
3. 在 VS Code 扩展中重新启动 CLIProxyAPI 服务。
4. 发起 Antigravity 登录，按浏览器页面完成 Google 授权。
5. 回到日志确认没有出现以下错误：

   ```text
   token exchange failed
   ```

此前的典型故障是 OAuth 授权页面可打开，但 CLIProxyAPI 在向 `https://oauth2.googleapis.com/token` 换取令牌时直连超时。配置 `proxy-url` 后，令牌交换也会走本地代理。

## 5. 在 Copilot Chat 中启用模型

登录成功后，在 VS Code 中：

1. 按 `Ctrl+Alt+I` 打开 Copilot Chat。
2. 点击模型选择器。
3. 选择 **Manage Models...**。
4. 手动启用所需模型，例如：
   - Gemini 3 Pro (Preview)
   - Gemini 3 Flash (Preview)
   - Claude Opus 4.5 (Thinking)
5. 点击模型旁的眼睛图标，使其可见并可选择。

模型启用需要在 VS Code UI 中手动完成。

## 常见故障速查

| 现象 | 原因与处理 |
| --- | --- |
| `dial tcp ... oauth2.googleapis.com ... timeout` | CLIProxyAPI 未走代理，检查 `config.yaml` 顶层的 `proxy-url`。 |
| `Test-NetConnection` 的 IPv4/IPv6 都失败 | 仅表示 Google 直连不可达；使用 `curl.exe -x` 验证代理出口。 |
| VS Code 出现 500 或 502 | 先确认 CLIProxyAPI 已成功登录、正在运行，并且 VS Code 与 CLIProxyAPI 都使用同一代理。 |
| 浏览器能授权，CLIProxyAPI 仍登录失败 | 浏览器与 CLIProxyAPI 的代理路径不同；以 `config.yaml` 的 `proxy-url` 为准。 |
| `curl.exe -x` 连接失败 | 代理软件未启动、端口不是 HTTP/Mixed 端口，或端口号填写错误。 |

## 已验证的关键配置

```yaml
# C:\\Users\\Administrator\\CLIProxyAPI\\config.yaml
proxy-url: "http://127.0.0.1:10888"
```

```json
// VS Code 用户 settings.json
{
  "http.proxy": "http://127.0.0.1:10888",
  "http.proxySupport": "override"
}
```

