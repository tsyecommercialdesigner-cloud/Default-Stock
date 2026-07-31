---
title: 解决Codex node runtime问题
created: 2026-07-31
tags:
  - 问题修复
---
**启动命令路径**和**环境变量名称**是关键线索：

- 启动命令路径：`...\AppData\Local\OpenAI\Codex\runtimes\cua_node\...\node_repl.exe`
    
- 环境变量包含：`BROWSER_USE_CODEX_APP_VERSION`、`NODE_REPL_TRUSTED_CODE_PATHS`、`SKY_CUA_NATIVE_PIPE` 等。
    

**这个 `node_repl` 既不是你手动配的，也不是 Cowart Canvas 插件自带的，而是 Codex 客户端官方（OpenAI / Codex Runtime）自身内置的原生环境服务。**

以下是它是如何被自动添加的完整逻辑：

## 1. 自动添加的完整过程

1. **Codex 官方 Runtime 的自动注入：**
    
    当你安装并运行 Codex 客户端时，Codex 会在后台自动下载并准备它所需的内置运行时（Runtime），也就是路径中的 `cua_node`（Computer Use Agent / Node 运行环境）。
    
2. **你运行了命令：`codex plugin add cowart@personal`**
    
    当你通过 Codex CLI 执行插件添加命令时，Codex 的底层系统被触发，**完成了两件事**：
    
    - **安装你指定的插件**：将 `cowart` 注册为插件服务（这就是之前列表里“来自插件”下的 `cowart_mcp`）。
        
    - **初始化/激活内部原生能力**：Codex 检测到插件系统已被激活，自动将它内部预设的原生 MCP 执行引擎——**`node_repl.exe`** 作为一个“内置服务器”自动挂载到你的个人配置文件（`C:\Users\Administrator\.codex\...`）中。
        
3. **环境变量的注入来源：**
    
    你看到的那些复杂的 ENV（环境变量），是 Codex 客户端在启动 `node_repl.exe` 时，**由 Codex 运行时（Runtime）自动向配置文件注入的内部参数**。
    
    - `CODEX_HOME` / `CODEX_CLI_PATH`：指定 Codex 的工作目录与 CLI 位置。
        
    - `NODE_REPL_TRUSTED_CODE_PATHS`：安全沙箱策略，指定允许运行代码的受信任路径（默认指向你的 `.codex` 目录）。
        
    - `BROWSER_USE_...` / `SKY_CUA_...`：用于 Codex 的网页浏览器控制（Computer Use / Browser Agent）和本地进程管道通信。
        

## 2. 为什么另一台电脑上没有？

如果另一台电脑上没有看到 `node_repl`，原因通常是：

- **Codex 版本差异**：另一台电脑上的 Codex 客户端版本可能较旧，或者没有启用 Computer Use / Node Runtime 等高级原生特性。
    
- **从未激活过原生 Runtime**：另一台电脑可能没有执行过特定的 CLI 命令，或者 Codex 尚未完成后台 Runtime（`cua_node`）的下载与释放。
    

## 总结

你执行的 `codex plugin add ...` 命令就像是一把“钥匙”，在安装 Cowart 插件的同时，也**顺带触发了 Codex 客户端内部原生引擎（`node_repl.exe`）的自动注册流程**。这些环境变量是 Codex 官方程序为你生成的内部运行上下文，你完全不需要手动修改或维护它们！

---
要让同事的电脑也正常显示并加载 `node_repl`（即 Codex 的原生 Node/Computer Use 运行时环境），你可以按以下步骤帮他排查和修复：

### 1. 触发 Codex 底层环境修复（推荐，最省力）

很多情况下，`node_repl` 没出来只是因为 Codex 后台没有完成 Runtime（`cua_node`）的解压和挂载。可以在同事的电脑上尝试**强制重置/重新安装一次插件**：

1. 打开终端（PowerShell 或 CMD），运行以下命令重新加载插件：
    
    Bash
    
    ```
    codex plugin remove cowart@personal
    codex plugin add cowart@personal
    ```
    
2. 完全退出 Codex 桌面客户端（注意：右下角系统托盘里的图标也要右键退出），然后重新打开 Codex。
    

### 2. 检查并强制更新 Codex 客户端版本

`node_repl.exe` 是 Codex 较新版本才引入的原生 Runtime。如果同事的 Codex 版本偏旧，就不会带有这个内置服务。

- **操作：** 检查同事电脑上的 Codex 是否有更新提示，将 Codex 客户端更新至最新版本。
    
- 更新完成后重新启动 Codex，系统会自动下载并释放 `cua_node` 运行环境。
    

### 3. 手动激活/检查 Runtime 文件是否存在

查看同事电脑的以下路径，确认 Codex 是否已经成功下载了 `cua_node` 运行时：

Plaintext

```
C:\Users\<同事用户名>\AppData\Local\OpenAI\Codex\runtimes\cua_node\
```

- **如果文件夹为空或不存在：** 说明 Codex 在安装或启动时网络受阻，未能成功下载官方 Runtime 组件。
    
    - **解决方法：** 检查网络代理/防火墙设置，确保 Codex 可以正常访问官方服务器拉取运行环境；或者直接将你电脑上 `AppData\Local\OpenAI\Codex\runtimes\` 目录下的 `cua_node` 文件夹复制到他电脑的相同位置。
        

### 4. 兜底方案：从你电脑导出 MCP 配置

如果上述自动修复都不生效，你可以直接把你电脑上生成的 `node_repl` 配置同步给他：

1. 在你的 Codex **插件/MCP 设置界面** 中，点击 `node_repl` 旁边的 **⚙️（齿轮图标）**。
    
2. 将启动命令路径（把路径里的用户名 `Administrator` 改成同事电脑的用户名）以及下方所有的环境变量复制出来。
    
3. 在同事电脑的 MCP 界面点击 **“+ 添加服务器”**，将这些配置手动填入并保存即可。

---

**提醒得非常及时！直接复制这几个值大概率是行不通的，千万不要全盘照抄。**

这里的环境变量可以分为 **“通用配置”** 和 **“设备/会话独占配置”** 两类。如果你把你电脑上的独占配置原封不动拷给他，会导致他那边的 `node_repl` 因为找不到对应的进程管道或校验失败而直接报错退出。

具体分析如下：

### 1. 绝对不能直接复制的字段（每台电脑或每次运行都不一样）

- **`SKY_CUA_NATIVE_PIPE_DIRECTORY`**（例如：`\\.\pipe\codex-computer-use-8c2414ff...`）
    
    - **原因：** 这是 Windows 上的命名管道（Named Pipe）路径。后半段的 `8c2414ff...` 是 Codex 每次启动时动态生成的随机 UUID/进程哈希，用于本地进程通信（IPC）。
        
    - **结果：** 跨电脑直接硬编码会导致同事的电脑找不到这个管道而连接超时。
        
- **`NODE_REPL_TRUSTED_BROWSER_CLIENT_SHA256S`**
    
    - **原因：** 这是本地浏览器客户端或扩展程序的 **SHA256 安全签名/指纹校验码**。如果同事电脑上的浏览器客户端版本或编译文件不一致，指纹就对不上。
        
- **包含 Hash 值的路径**（例如：`cua_node\f8d2abcb7481383b\...`）
    
    - **原因：** 路径里的 `f8d2abcb7481383b` 是 Codex 下载的 Runtime 版本 Hash。不同版本的 Codex 下载下来的文件夹名字是不同的。
        

### 2. 可以通用复制的字段

- **功能指令类：**
    
    - `NODE_REPL_INSTRUCTIONS_USE_CASE_BROWSER`
        
    - `NODE_REPL_INSTRUCTIONS_USE_CASE_CHROME`
        
    - `BROWSER_USE_AVAILABLE_BACKENDS` (`chrome,iab`)
        
    - `SKY_CUA_NATIVE_PIPE` (`1`)
        
    - `NODE_REPL_NATIVE_PIPE_CONNECT_TIMEOUT_MS` (`1000`)
        
- **基本路径类（只需更改用户名）：**
    
    - `CODEX_HOME` (`C:\Users\<同事用户名>\.codex`)
        

### 3. 正确的解决思路

既然涉及到动态生成的值，**最稳妥、最标准的做法绝对不是手动配置 JSON**，而是**让 Codex 自动生成属于他那台电脑的正确配置**：

1. **检查同事电脑上到底有没有 `node_repl.exe` 文件**：
    
    让他打开路径：`C:\Users\<同事用户名>\AppData\Local\OpenAI\Codex\runtimes\cua_node\`
    
    - 如果该目录下**什么都没有**：说明他的 Codex **根本没有下载完官方 Runtime**（通常是因为网络代理拦住了 OpenAI 的静态资源下载）。解决网络问题或把你的 `cua_node` 文件夹整个拷给他，Codex 重新识别后就会自动生成正确的管道和 Hash 变量。
        
    - 如果**已有该文件**：直接在 Codex 中彻底**登出账号并重新登录**，或**重装 Codex 客户端**，系统就会自动在后台将 `node_repl` 重新注册进界面，完全不需要人工手动填写这些复杂的 ENV。