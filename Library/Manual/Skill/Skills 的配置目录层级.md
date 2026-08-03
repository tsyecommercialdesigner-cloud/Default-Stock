---
title: Skills 的配置目录层级
created: 2026-08-03
tags:
  - Skills
  - Agent_Skills
  - Skills_Path
---
---
*本文介绍 Windows 11 环境下 Codex 与 Claude Code 的 Skill、插件及管理配置目录之间的区别。*

## 一、先区分三个概念

在讨论目录前，应先区分：

1. **用户级 Skill**：供当前操作系统用户使用。
2. **项目级 Skill**：只对某个项目或代码仓库生效。
3. **插件级 Skill**：随插件一起安装，位于插件包内部。
4. **管理员或企业级配置**：由系统管理员统一部署，通常用于强制策略、MCP 配置或安全设置；它不一定等同于“企业级 Skill 目录”。

---

# 二、Codex 的 Skill 目录

## 2.1 用户级 Skill

Windows 11 中，用户级 Skill 的推荐目录为：

```text
%USERPROFILE%\.agents\skills\
```

展开后通常类似：

```text
C:\Users\<用户名>\.agents\skills\
```

示例：

```text
C:\Users\SiYuan\.agents\skills\my-skill\
├─ SKILL.md
├─ scripts\
├─ references\
├─ assets\
└─ agents\
   └─ openai.yaml
```

每个 Skill 通常拥有独立目录，并以 `SKILL.md` 作为核心说明文件。

---

## 2.2 项目级 Skill

项目级 Skill 放在项目目录下：

```text
<项目目录>\.agents\skills\
```

例如：

```text
D:\Projects\my-app\.agents\skills\deploy\SKILL.md
```

Codex 可以从当前工作目录向上查找 `.agents\skills`，直到项目或仓库的上级边界。

项目级 Skill 适合保存：

- 项目专属构建流程
- 仓库代码规范
- 测试或部署步骤
- 团队协作约定
- 只适用于当前代码库的自动化指令

---

## 2.3 `.codex\skills` 与旧安装方式

部分旧版安装器、脚本或历史实现可能将 Skill 安装到：

```text
%USERPROFILE%\.codex\skills\
```

即：

```text
C:\Users\<用户名>\.codex\skills\
```

因此，在排查 Skill 时，可以同时检查：

```powershell
Get-ChildItem "$env:USERPROFILE\.agents\skills" -Force
Get-ChildItem "$env:USERPROFILE\.codex\skills" -Force
```

一般应优先把 `.agents\skills` 视为当前用户手动维护 Skill 的主要位置；`.codex\skills` 更可能与旧版安装器或特定实现有关。

---

## 2.4 Codex 插件级 Skill

“插件级 Skill”不一定直接平铺在用户 Skill 目录中。

更准确地说：

```text
<插件安装目录>\<插件包内部的 Skill 目录>
```

插件的实际安装位置可能由应用、插件管理器或运行环境维护。插件包内部可以携带：

- Skill
- MCP Server
- Agent 配置
- 脚本
- 资源文件
- 插件元数据

因此，不能仅凭 Skill 名称推断它一定在：

```text
%USERPROFILE%\.agents\skills
```

或：

```text
%USERPROFILE%\.codex\skills
```

用户手动创建和维护的 Skill，与插件系统自动安装的 Skill，应分别理解。

---

## 2.5 Codex 的层级与同名 Skill

Codex 可以从多个作用域发现 Skill，例如：

```text
项目级 / 用户级 / 管理员级 / 系统内置
```

但“发现层级”不一定等于“覆盖优先级”。

当两个 Skill 使用相同名称时，不应简单假定：

```text
项目级一定覆盖用户级
```

或：

```text
管理员级一定覆盖项目级
```

更安全的做法是给不同来源的 Skill 使用明确名称，例如：

```yaml
name: company-deploy
```

和：

```yaml
name: personal-deploy
```

这样可以避免同名 Skill 同时被发现时产生歧义。

---

# 三、Claude Code 的 Skill 和插件目录

## 3.1 用户级 Skill

Claude Code 的用户级 Skill 通常位于：

```text
~/.claude/skills/
```

Windows 11 中对应：

```text
%USERPROFILE%\.claude\skills\
```

示例：

```text
C:\Users\<用户名>\.claude\skills\review\SKILL.md
```

---

## 3.2 项目级 Skill

项目级 Skill 通常位于：

```text
<项目目录>\.claude\skills\
```

示例：

```text
D:\Projects\my-app\.claude\skills\review\SKILL.md
```

它只随项目使用，适合保存代码仓库专属规则和工作流。

---

## 3.3 插件管理目录

Claude Code 的插件通常由以下目录管理：

```text
~/.claude/plugins/
```

Windows 11 中对应：

```text
%USERPROFILE%\.claude\plugins\
```

但应注意：

> `plugins` 是插件管理目录，不是把所有插件级 Skill 直接平铺在其中的 Skill 根目录。

插件级 Skill 通常位于具体插件包内部，例如：

```text
~/.claude/plugins/
└─ <插件相关目录>/
   └─ my-plugin/
      ├─ .claude-plugin/
      │  └─ plugin.json
      ├─ skills/
      │  └─ review/
      │     └─ SKILL.md
      ├─ agents/
      ├─ commands/
      └─ hooks/
```

因此，插件级 Skill 的逻辑路径为：

```text
<plugin-root>/skills/<skill-name>/SKILL.md
```

可以在 PowerShell 中递归查找：

```powershell
Get-ChildItem "$env:USERPROFILE\.claude\plugins" -Recurse -Filter SKILL.md
```

---

# 四、企业级配置与企业级 Skill 的区别

## 4.1 Claude Code 的 Managed 配置

Windows 中可能存在：

```text
C:\Program Files\ClaudeCode\
```

这里更适合称为：

```text
企业或管理员 Managed 配置位置
```

例如其中可能包含：

```text
managed-settings.json
managed-mcp.json
```

Managed 配置可用于统一控制：

- 安全策略
- MCP Server
- 功能开关
- 组织级限制
- 管理员强制设置

这些配置可能具有高于用户配置和项目配置的优先级。

但是：

> `C:\Program Files\ClaudeCode\` 不应直接等同于一个通用的“企业级 Skills 根目录”。

“管理员配置优先级”与“多个 Skill 的发现和同名处理”是两个不同问题。

---

## 4.2 Codex 是否有类似机制

Codex 同样可以存在：

- 项目级资源
- 用户级资源
- 管理员部署资源
- 系统内置资源
- 管理策略

但不能直接把 Claude Code 的 Windows 目录规则照搬为：

```text
C:\Program Files\Codex\skills
```

除非当前 Codex 版本或组织部署文档明确规定了该路径。

对于 Windows 原生环境，应以当前版本文档、安装器行为和实际目录为准；对于 WSL、Linux 或容器环境，管理员级目录可能采用类 Unix 路径。

---

# 五、Codex 与 Claude Code 对照表

| 类型 | Codex | Claude Code |
|---|---|---|
| 用户级 Skill | `%USERPROFILE%\.agents\skills\` | `%USERPROFILE%\.claude\skills\` |
| 项目级 Skill | `<项目>\.agents\skills\` | `<项目>\.claude\skills\` |
| 旧版或特定安装器目录 | `%USERPROFILE%\.codex\skills\` | 视插件或版本而定 |
| 插件管理目录 | 由 Codex/插件系统管理 | `%USERPROFILE%\.claude\plugins\` |
| 插件级 Skill | 位于具体插件包内部 | `<plugin-root>/skills/<skill>/SKILL.md` |
| 企业 Managed 配置 | 与安装方式和管理员策略有关 | Windows 下可能位于 `C:\Program Files\ClaudeCode\` |
| 同名 Skill | 不宜假定自动覆盖，应避免重名 | 应结合具体作用域和版本规则判断 |

---

# 六、排查目录的实用命令

## 6.1 检查 Codex 用户 Skill

```powershell
Get-ChildItem "$env:USERPROFILE\.agents\skills" -Force
```

## 6.2 检查旧 Codex Skill 目录

```powershell
Get-ChildItem "$env:USERPROFILE\.codex\skills" -Force
```

## 6.3 查找 Claude Code 用户级 Skill

```powershell
Get-ChildItem "$env:USERPROFILE\.claude\skills" -Recurse -Filter SKILL.md
```

## 6.4 查找 Claude Code 插件中的 Skill

```powershell
Get-ChildItem "$env:USERPROFILE\.claude\plugins" -Recurse -Filter SKILL.md
```

## 6.5 在某个项目中查找全部 Skill

```powershell
Get-ChildItem "D:\Projects\my-app" -Recurse -Filter SKILL.md
```

---
# 七、其它 Harness 的 Skill 所在目录

## 7.1 Antigravity

### 用户级 Skill

Antigravity 的用户级 Skill 通常位于：

```text
~/.gemini/antigravity/skills/
```

Windows 11 中对应：

```text
%USERPROFILE%\.gemini\antigravity\skills\
```

示例：

```text
C:\Users\<用户名>\.gemini\antigravity\skills\review\SKILL.md
```

---

### 项目级 Skill

项目级 Skill 通常位于：

```text
<项目目录>\.agent\skills\
```

示例：

```text
D:\Projects\my-app\.agent\skills\review\SKILL.md
```

它只随项目使用，适合保存代码仓库专属规则和工作流。

---
# 八、结论

记忆这些核心路径即可：

```text
Codex 用户级：
%USERPROFILE%\.agents\skills\

Codex 项目级：
<项目>\.agents\skills\

Claude Code 用户级：
%USERPROFILE%\.claude\skills\

Claude Code 项目级：
<项目>\.claude\skills\

Claude Code 插件管理：
%USERPROFILE%\.claude\plugins\
```

同时牢记：

> 插件管理目录、插件包内部的 Skill、用户级 Skill、项目级 Skill和管理员 Managed 配置是不同概念，不能混为一谈。
