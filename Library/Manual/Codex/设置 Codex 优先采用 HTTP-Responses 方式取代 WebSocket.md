---
title: 设置Codex优先采用 HTTP 方式取代 WebSocket
created: 2026-07-13
tags:
  - Codex
  - Codex_reconnecting
source_url: https://mp.weixin.qq.com/s/E_prb92-x4rjdbv3Oj7opA
---
---
## 一、问题现象与本质

### 1.1 表现：一直卡在“重新连接 5/5”

典型现象：

- 终端里不断出现：正在重新连接 1/5 → 2/5 → 3/5 → 4/5 → 5/5  

绝大多数人以为是「模型太慢」，其实很多时候是**连接层在疯狂重试 WebSocket**，而不是模型在深度推理。[page:1]

### 1.2 默认连接策略（旧路径）

默认链路大致是：

- Codex → 建立 WebSocket → 失败 → 重连 1/5 → … → 重连 5/5 →  
- 最后才降级到 HTTP，再开始正常对话。[page:1]

一旦你的网络环境对 WebSocket 不友好，就会频繁卡在这个重试环节。[page:1]

---

## 二、核心思路：直接走 HTTP / Responses

**目标：**  
不要让 Codex 先死磕 WebSocket，而是直接使用一个 HTTP / Responses 的 provider。[page:1]

- 旧路径：Codex → WebSocket → 失败 × 5 → 降级 HTTP → 开始工作  
- 新路径：Codex → HTTP / Responses → 直接开始工作。[page:1]

只要你的网络本身更适合普通 HTTPS 请求，这个调整通常能明显减少「重新连接 5/5」等待时间。[page:1]

---

## 三、实操步骤

> 建议按顺序完成：找到配置 → 备份 → 添加 provider → 重启测试。[page:1]

### 3.1 步骤一：找到 `config.toml`

Codex 的配置文件位置如下。[page:1]

**Windows：**

- 路径示例：  
  `C:\Users\你的用户名\.codex\config.toml`  
- 也可以在资源管理器地址栏输入：  
  `%USERPROFILE%\.codex`  
  然后找到 `config.toml`。[page:1]

**macOS / Linux：**

- 通常在：  
  `~/.codex/config.toml`。[page:1]

如果不确定路径，可以在 Codex 里发一句提示让它帮你定位（只读）：[page:1]

```text
请帮我定位当前 Codex 的 config.toml 配置文件路径，只告诉我路径，不要修改文件。
```

---

### 3.2 步骤二：备份配置文件

强烈建议先备份，再修改。[page:1]

参考做法：

- 在同目录复制一份：`config.toml.bak`  
- 或者把原内容复制到一个单独的文本文件里保存。[page:1]

---

### 3.3 步骤三：添加 HTTP Provider 配置

在 `config.toml` 中**新增**一个 provider，而不是删除原有配置。[page:1]

在合适位置加入如下内容（示例）：[page:1]

```toml
model_provider = "openai_http"

[model_providers.openai_http]
name = "OpenAI HTTP"
wire_api = "responses"
requires_openai_auth = true
supports_websockets = false
```

含义说明：

- `model_provider = "openai_http"`  
  将默认 provider 设为 `openai_http`。[page:1]
- `wire_api = "responses"`  
  使用 Responses API 模式（走普通 HTTPS 请求）。[page:1]
- `requires_openai_auth = true`  
  继续沿用 OpenAI / ChatGPT 登录授权方式。[page:1]
- `supports_websockets = false`  
  告诉 Codex：不要优先使用 WebSocket。[page:1]

注意要点：

- 不要随便删除你原来的 provider 配置，只新增这个块即可。[page:1]
- 确保顶部 `model_provider` 的值与下面 `model_providers.xxx` 的名字一一对应：这里必须都叫 `openai_http`。[page:1]

---

### 3.4 步骤四：重启 Codex 并测试

1. 保存 `config.toml`。  
2. 关闭当前所有 Codex 终端会话。[page:1]  
3. 新开终端，重新启动 Codex：

```bash
codex
```

4. 发一条简单消息进行测试：[page:1]

```text
请用一句话回复：Codex 连接测试成功。
```

如果此时不再出现长时间的「重新连接 5/5」，说明配置在你的网络环境里是有效的。[page:1]

---

## 四、给“小白”用的自动修改提示词

如果你不想手动改配置，可以把下面这段提示词直接丢给 Codex，让它执行（关键是先备份后修改）。[page:1]

```markdown
请帮我检查并修改 Codex 配置，目标是减少“正在重新连接 5/5”的问题。
要求：
1. 先定位我的 ~/.codex/config.toml 文件；
2. 先备份为 config.toml.bak；
3. 不要删除我已有配置；
4. 新增一个名为 openai_http 的 provider；
5. 配置 wire_api = "responses"；
6. 配置 supports_websockets = false；
7. 如果我使用的是 ChatGPT / OpenAI 登录授权，请保留 requires_openai_auth = true；
8. 修改完成后告诉我改了哪些内容；
9. 最后让我重启 Codex 测试。
```

---

## 五、如果改完仍然卡住

> 这个方案不是 100% 对所有情况都生效，改完还卡时可以继续排查。[page:1]

### 5.1 升级 Codex 版本

确保不是旧版本问题。[page:1]

```bash
npm install -g @openai/codex@latest
```

如果不是通过 `npm` 安装，按你原来的安装方式更新即可。[page:1]

---

### 5.2 检查代理 / 网络对 WebSocket 的支持

很多代理和网络对网页浏览友好，但对 WebSocket 不稳定。[page:1]

可以尝试：

- 更换代理节点  
- 开启全局代理  
- 使用明确支持 WebSocket 的线路  
- 暂时关闭可能拦截连接的安全软件  
- 避免在公司网络 / 严格防火墙环境使用。[page:1]

如果换网络后立刻恢复正常，多半就是链路问题。[page:1]

---

### 5.3 新开会话测试

旧会话可能持有旧的 provider 或上下文缓存。[page:1]

建议：

- 关闭当前会话  
- 回到项目目录，重新启动 Codex 再测一次。[page:1]

---

### 5.4 检查配置是否真的生效

重点检查以下两处是否一致：[page:1]

```toml
model_provider = "openai_http"
```

```toml
[model_providers.openai_http]
wire_api = "responses"
supports_websockets = false
```

常见错误：

- `model_provider = "openai_http"`  
  但下面写成 `[model_providers.chatgpt_http]`，名字对不上就不会生效。[page:1]

---

## 六、改完后的体验预期

改成 HTTP / Responses 后，Codex 仍然可能偶尔卡顿，但**概率会明显降低**。[page:1]

大致体验：

- 单轮问答：几乎感觉不到差异  
- 短对话（5–10 轮）：偶尔多几十毫秒，基本无感  
- 长会话（几十轮以上）：偶尔重连，但不会再死锁在「重新连接 5/5」。[page:1]

如果你的工作流是「问完一个问题就结束」，对体验几乎没有负面影响。[page:1]

---

## 七、真实案例（网络环境导致的奇怪问题）

几个典型场景：[page:1]

- 某大厂程序员：  
  公司统一代理策略中，WebSocket 被特殊路由，HTTPS 仍走国内链路，导致 WebSocket 经常失败，但降级 HTTP 后秒回；改成 `supports_websockets = false` 后问题消失。[page:1]

- 某上市公司 DBA：  
  笔记本常年挂 Clash，WebSocket 走 SOCKS5，稳定性较差；切 HTTP 后明显稳定。[page:1]

- 自由职业者：  
  家庭宽带出口 IP 被 OpenAI 限流，WebSocket 几乎连不上；改完 HTTP 配置后恢复正常使用。[page:1]

共性：网络环境普遍「不友好 WebSocket」，而 Codex 默认又优先 WebSocket，所以容易卡。[page:1]

---

## 八、第三方模型的情况

使用 OpenRouter、Azure、自建网关等第三方模型时，这套思路仍然适用，只是配置会略有不同。[page:1]

### 8.1 OpenRouter 示例

需要指定 `base_url`：[page:1]

```toml
[model_providers.openrouter_http]
name = "OpenRouter HTTP"
base_url = "https://openrouter.ai/api/v1"
wire_api = "responses"
requires_openai_auth = false
```

### 8.2 Azure / 自建网关

- Azure OpenAI：路径和认证方式不同，需要参考 Azure 文档配置。[page:1]  
- 自建网关：只要兼容 OpenAI 的 Responses API，就可以沿用「不走 WebSocket、只用 HTTP/Responses」这一思路。[page:1]

核心仍然是：**关闭 WebSocket 优先策略，走 HTTP / Responses。**[page:1]

---

## 九、作者踩过的几个坑（可以自查）

1. 复制粘贴时丢了反斜杠  
   - TOML 中 Windows 路径需要双反斜杠或改用正斜杠，否则 Codex 启动报错。[page:1]

2. provider 名称不一致  
   - 顶部写 `model_provider = "openai_http"`  
   - 下面却写 `[model_providers.chatgpt_http]`，导致配置失效。[page:1]

3. 改完配置没重启  
   - 保存了 `config.toml`，但 Codex 进程没重启，新配置不会生效。[page:1]

4. 备份文件放错地方  
   - 把 `config.toml.bak` 丢到桌面，后续清理文件时一起删掉了。  
   - 建议跟原配置放在同目录，或放在项目仓库里便于版本管理。[page:1]

---

## 十、完整 Checklist（操作前后自检）

### 10.1 改之前

- [ ] 确认 Codex 已是最新版  
- [ ] 找到 `config.toml` 的准确路径  
- [ ] 备份为 `config.toml.bak`。[page:1]

### 10.2 改的时候

- [ ] 只新增 provider，不删除原有配置  
- [ ] 顶部 `model_provider` 与 `[model_providers.xxx]` 名称一致  
- [ ] `wire_api = "responses"` 已配置  
- [ ] `supports_websockets = false` 已配置  
- [ ] `requires_openai_auth = true` 是否与实际登录方式匹配。[page:1]

### 10.3 改之后

- [ ] 保存 `config.toml`  
- [ ] 关闭所有 Codex 终端  
- [ ] 重新启动 `codex`  
- [ ] 发一条简单消息，确认不会再卡在「重新连接 5/5」。[page:1]

---

## 十一、版本变化与后续关注

Codex 的连接策略会持续更新，WebSocket 稳定性和降级逻辑也会不断调整。[page:1]

建议习惯：

- 关注 Codex changelog  
- 碰到新卡顿先去 GitHub Issues 查是否为已知问题  
- 网络环境不佳时，以 HTTP/Responses 兜底能解决大部分问题  
- 如果仍无法解决，可以提 issue，附上：网络环境描述、Codex 版本号、错误日志等。[page:1]

> 工具的目的，是帮我们节省时间，而不是多一个需要“对付”的对象。  
> 把连接层稳定下来，你就能把精力更多放在真正想做的事上。[page:1]