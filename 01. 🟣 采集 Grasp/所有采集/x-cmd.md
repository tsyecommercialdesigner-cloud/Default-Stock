**一句话总结：**  
GitHub 上的 **x-cmd/x-cmd** 是一个让你在终端里“一键拥有上千个命令行工具”的 **超级 Shell 工具箱**，同时也是专为 **AI Agents（Claude Code、OpenClaw、Cursor 等）增强能力** 的“终端外挂”。它能让你的 Bash/Zsh 变得像 Python 标准库一样强大。[^1][^2]

---

# 🧩 x-cmd 是什么？（萌新版解释）

x-cmd 就像是：

- **你的终端外挂**：让原本简陋的 Bash/Zsh 拥有 300+ 内置模块（文件处理、主题、Git、系统管理、AI 调用……）[^1]。
    
- **你的命令行 App Store**：600+ CLI 工具随用随装（jq、fzf、ripgrep、fd、node、python…）[^1]。
    
- **你的 AI Agent 超能力库**：让 Claude Code、OpenClaw、Cursor 等 AI 直接调用上千命令[^3]。
    
- **你的开发环境管理器**：Node/Python/Java/Go 一键装好[^4]。
    
- **你的终端美化器**：主题、字体、导航、补全一条命令搞定[^3]。
    
- **你的安全沙箱**：可在隔离环境里运行不放心的软件[^3]。
    

它的核心只有 **1.1MB**，加载速度 **20–60ms**，非常轻量[^1]。

---

# 🚀 萌新如何开始使用 x-cmd？

## 1. 安装（最简单方式）

在 Bash / Zsh 输入：

```
eval "$(curl https://get.x-cmd.com)"
```

或：

````
eval "$(wget -O- https://get.x-cmd.com)"
```[^1]


安装后你会得到一个命令：  
**`x`**

***

# 🧪 萌新最应该学的 5 个用法（从简单到强）

## ① **[安装工具](ca://s?q=教我如何用_x_env_use_安装工具)**：像 App Store 一样
```bash
x env use jq
x env use fzf
x env use python
````

这会自动下载并配置好工具，不污染系统[^4]。

---

## ② **运行模块**：300+ 内置功能

例如：

````
x theme        # 切换终端主题
x advise       # 自动补全
x ccal         # 中国农历日历
x openai       # 调用 OpenAI API
x gemini       # 调用 Google Gemini
x deepseek     # 调用 DeepSeek
```[^1]


***

## ③ **[AI Agent 超能力](ca://s?q=如何让我的_AI_Agent_使用_x_cmd)**  
如果你用 Claude Code / Cursor / OpenClaw：

```bash
. ~/.x-cmd.root/X
````

AI 就能调用上千命令行工具（官方称为“Shell Superpowers for AI Agents”）[^1]。

---

## ④ **快速试用工具**（不安装）

```
x env try node
```

临时使用 Node，不会写入系统[^4]。

---

## ⑤ **清理空间**

```
x env gc
```

清理下载的工具缓存[^4]。

---

# 🧭 萌新学习路线（最简单的 7 步）

1. **安装 x-cmd**
    
2. **学会 `x env use` 安装工具**
    
3. **学会 `x env ls` 查看已安装工具**
    
4. **学会 `x env try` 临时试用工具**
    
5. **学会几个常用模块：`x theme`、`x advise`、`x openai`**
    
6. **把 `. ~/.x-cmd.root/X` 加到你的 AI Agent 里**
    
7. **开始用 AI + x-cmd 自动化你的电脑**
    

---

# 📦 常用命令速查表（萌新必收藏）

|功能|命令|说明|
|---|---|---|
|**安装工具**|`x env use <pkg>`|安装 jq、fzf、python 等|
|**查看工具**|`x env ls`|列出已安装工具|
|**临时试用**|`x env try <pkg>`|不污染系统|
|**卸载工具**|`x env unuse <pkg>`|卸载|
|**清理空间**|`x env gc`|清理缓存|
|**切换主题**|`x theme`|美化终端|
|**AI 调用**|`x openai`|调用 OpenAI API|

---

# 🧙 为什么 x-cmd 特别适合你（萌新视角）

- 不需要懂 Linux，也能用强大的命令行工具
    
- 不需要配置环境，Node/Python/Java 一键装好
    
- 不需要懂 AI API，`x openai` 就能直接用
    
- 不需要折腾主题，`x theme` 一键美化
    
- 不需要担心系统被污染，所有工具都在 x-cmd 的沙箱里
    

你只要记住一个命令：  
**`x`**

---

# 🎯 下一步你想让我帮你做什么？

你可以点下面任何一个继续深入：

- 帮我安装 x-cmd
    
- 告诉我最适合萌新的 x-cmd 入门教程
    
- 教我用 x-cmd 调用 OpenAI/DeepSeek
    
- 帮我用 x-cmd 配好 Python/Node 环境
    

思缘，你想从哪一步开始？

---

# References (4)

[^1]: _x-cmd/README.md at X · x-cmd/x-cmd · GitHub_. [https://github.com/x-cmd/x-cmd/blob/X/README.md](https://github.com/x-cmd/x-cmd/blob/X/README.md)

[^2]: _GitHub - x-cmd/x-cmd: Best Buddy for AI Agents. Bootstrap 1000+ command line tools in seconds. · GitHub_. [https://github.com/x-cmd/x-cmd?tab=Apache-2.0-1-ov-file](https://github.com/x-cmd/x-cmd?tab=Apache-2.0-1-ov-file)

[^3]: _GitHub - x-cmd/x-cmd at terminaltrove · GitHub_. [https://github.com/x-cmd/x-cmd?ref=terminaltrove](https://github.com/x-cmd/x-cmd?ref=terminaltrove)

[^4]: _GitHub - forkgitss/x-cmd-x-cmd: x-cmd: Bootstrap 1000+ command line tools in seconds._. [https://github.com/forkgitss/x-cmd-x-cmd](https://github.com/forkgitss/x-cmd-x-cmd)