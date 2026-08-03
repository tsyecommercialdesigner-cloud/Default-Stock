---
title: MCP配置与故障排查指南
created: 2026-08-03
tags:
  - MCP
  - MCP_Server
  - MCP_Config
---
---
# MCP 配置文件

## MCP 配置文件的所在目录

### 用户级

所有MCP Server 的设置都被存储在该文件中，其所在目录如下。

Windows：

```plain
C:\Users\<用户名>\.gemini\antigravity\mcp_config.json
```

macOS/Linux：

```plain
~/.gemini/antigravity/mcp_config.json
```

### 项目级

目前 Antigravity 的 MCP 配置主要是用户级，但可以通过以下方式达成项目级的效果：

#### 方法一：使用 AGENTS.md 指定 MCP 使用

在项目根目录建立 `AGENTS.md` 文件，告诉 AI 在这个项目中优先使用哪些 MCP：

```markdown
# Project Instructions

## MCP Usage
- 本项目使用 Firebase 作为后端，请优先使用 Firebase MCP
- 数据库操作请使用 PostgreSQL MCP
- 不要使用 Supabase MCP
```

---
#### 方法二：环境变量切换

在 `mcp_config.json` 中使用环境变量，然后在不同项目中设置不同的值：

```json
{
  "mcpServers": {
    "postgres": {
      "command": "postgres-mcp",
      "env": {
        "DATABASE_URI": "${PROJECT_DATABASE_URI}"
      }
    }
  }
}
```

然后在项目的 `.env` 或启动脚本中设置：

```bash
export PROJECT_DATABASE_URI="postgresql://user:pass@localhost:5432/myproject"
```

---
## MCP 配置文件的格式

### 一般格式

`mcp_config.json` 使用 JSON 格式，基本结构如下：

```json
{
  "mcpServers": {
    "server-name": {
      "command": "执行指令",
      "args": ["参数1", "参数2"],
      "env": {
        "API_KEY": "你的 API Key"
      }
    }
  }
}
```

### 示例：GitHub MCP Server 设置

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "你的 GitHub Token"
      }
    }
  }
}
```

### 示例：Context7 MCP Server 设置（HTTP 传输）

```json
{
  "mcpServers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp"
    }
  }
}
```

---
## 敏感信息的最佳实践

>[!warning] 注意
>不管是全局还是项目层级，都不要把敏感信息直接写在配置文件中！

**建议做法**：在 Shell 设定档（`~/.zshrc` 或 `~/.bashrc`）中设定环境变量：

```bash
# MCP 相关的环境变量
export GITHUB_TOKEN="ghp_xxxxxxxxxxxx"
export POSTGRES_URL="postgresql://..."
export SLACK_TOKEN="xoxb-..."
export OPENAI_API_KEY="sk-..."
```

缓存在 `mcp_config.json` 中引用：

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

社区已经在讨论 per-workspace 的 MCP 设定功能，未来 Antigravity 可能会原生支援项目层级的 MCP 设定。可以关注 Google AI Developers Forum [<sup>1</sup>](https://discuss.ai.google.dev/) 的最新动态。

---
# MCP 故障排查

## MCP Server 无法连接

1. 确认已安装 Node.js 和 npm
2. 确认配置文件中的路径是绝对路径
3. 重新启动 Antigravity

## Windows 特别注意事项

在 Windows 上使用 stdio 传输的 MCP Server 时，需要使用 cmd/c 包装：

```json
{
  "mcpServers": {
    "my-server": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@some/package"]
    }
  }
}
```

## 环境变量无法读取

确认环境变量的格式正确：

```json
{
  "env": {
    "API_KEY": "${MY_API_KEY}",
    "DEBUG": "${DEBUG:-false}"
  }
}
```

支持的语法：

- `${VAR}`：基本变量替换
- `${VAR:-default}`：设置默认值
- `${VAR:?error}`：必需变量检查

---
