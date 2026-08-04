---
title: Lovart 接入 Codex-STDIO-MCP
created: 2026-08-01
tags:
  - MCP_Server
  - Lovart
---
---
# Lovart 接入 Codex：官方 Skill + 本地 STDIO MCP SOP

## 目标

让 Codex 能通过本机的 STDIO MCP 调用 Lovart，安全地生成或编辑图片、视频、音频和 3D 内容。

推荐架构：

```text
Codex → 本地 STDIO MCP → Lovart 官方 agent_skill.py → Lovart Agent OpenAPI
```

> 不在 MCP 中重写 Lovart 的 HMAC 签名或 API 地址。MCP 仅转发到官方脚本，认证、轮询、下载逻辑继续由官方 skill 维护。

---

## 前置条件

- Windows 已安装 Python（命令为 `python`）。
- 已安装 Codex Desktop。
- 已有 Lovart 账号，并能在 Lovart 网页的 **Avatar → AK/SK Management** 获取 AK/SK。

**安全要求：** 不要在聊天、Markdown、Python 源码或 `config.toml` 中记录 AK/SK。

---

## 1. 安装官方 Lovart skill

在 Git Bash 执行：

```bash
cd ~/.codex/skills
git clone https://github.com/lovartai/lovart-skill.git
```

如果已克隆过（本机当前即为此情况），不要重复克隆；需要更新时执行：

```bash
cd ~/.codex/skills/lovart-skill
git pull
```

无需移动文件。官方脚本的固定位置为：

```text
C:\Users\Administrator\.codex\skills\lovart-skill\skills\lovart-skill\scripts\agent_skill.py
```

---

## 2. 配置 Lovart 凭据

在 Git Bash 执行，替换为真实密钥：

```bash
setx LOVART_ACCESS_KEY "ak_xxx"
setx LOVART_SECRET_KEY "sk_xxx"
```

关闭并重新打开 Git Bash 后，验证密钥是否可被官方脚本使用：

```bash
python ~/.codex/skills/lovart-skill/skills/lovart-skill/scripts/agent_skill.py config --json
```

> `setx` 写入的是后续新启动程序可见的用户环境变量；它不会更新当前已打开的终端或已运行的 Codex。

---

## 3. 创建本地 MCP 运行环境

在 Git Bash 执行：

```bash
mkdir -p ~/.codex/mcp/lovart
cd ~/.codex/mcp/lovart

python -m venv .venv
./.venv/Scripts/python.exe -m pip install -U "mcp>=1,<2"
```

---

## 4. 创建 STDIO MCP 服务

创建文件：

```text
C:\Users\Administrator\.codex\mcp\lovart\server.py
```

写入以下内容：

```python
from __future__ import annotations

import json
import os
import subprocess
import sys
from pathlib import Path
from typing import Any, Literal

from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Lovart")

SKILL_SCRIPT = Path(
    os.getenv(
        "LOVART_SKILL_SCRIPT",
        str(
            Path.home()
            / ".codex"
            / "skills"
            / "lovart-skill"
            / "skills"
            / "lovart-skill"
            / "scripts"
            / "agent_skill.py"
        ),
    )
)


def run_lovart(args: list[str], timeout: int = 900) -> dict[str, Any]:
    if not SKILL_SCRIPT.is_file():
        raise RuntimeError(f"找不到 Lovart 官方脚本：{SKILL_SCRIPT}")

    if not os.getenv("LOVART_ACCESS_KEY") or not os.getenv("LOVART_SECRET_KEY"):
        raise RuntimeError("未检测到 LOVART_ACCESS_KEY 或 LOVART_SECRET_KEY。")

    result = subprocess.run(
        [sys.executable, str(SKILL_SCRIPT), *args],
        capture_output=True,
        text=True,
        encoding="utf-8",
        timeout=timeout,
        env=os.environ.copy(),
    )

    stdout = result.stdout.strip()
    stderr = result.stderr.strip()
    if result.returncode != 0:
        raise RuntimeError(stderr or stdout or "Lovart 调用失败。")

    try:
        return json.loads(stdout)
    except json.JSONDecodeError:
        return {"output": stdout}


@mcp.tool()
def lovart_projects() -> dict[str, Any]:
    """列出当前 Lovart 账号可用项目。"""
    return run_lovart(["projects", "--json"], timeout=60)


@mcp.tool()
def lovart_threads(all_threads: bool = False) -> dict[str, Any]:
    """列出本地保存的 Lovart 对话线程。"""
    args = ["threads", "--json"]
    if all_threads:
        args.insert(1, "--all")
    return run_lovart(args, timeout=60)


@mcp.tool()
def lovart_generate(
    prompt: str,
    project_id: str | None = None,
    thread_id: str | None = None,
    reasoning_mode: Literal["fast", "thinking"] = "fast",
) -> dict[str, Any]:
    """通过 Lovart 生成或编辑图片、视频、音频或 3D 内容，并下载结果。"""
    args = [
        "chat", "--prompt", prompt, "--mode", reasoning_mode,
        "--json", "--download",
    ]
    if project_id:
        args += ["--project-id", project_id]
    if thread_id:
        args += ["--thread-id", thread_id]
    return run_lovart(args, timeout=900)


@mcp.tool()
def lovart_result(thread_id: str) -> dict[str, Any]:
    """获取一个 Lovart 线程的最新生成结果，并下载可用素材。"""
    return run_lovart(
        ["result", "--thread-id", thread_id, "--json", "--download"],
        timeout=900,
    )


@mcp.tool()
def lovart_confirm(thread_id: str) -> dict[str, Any]:
    """确认高费用任务。仅在用户明确确认费用后调用。"""
    return run_lovart(
        ["confirm", "--thread-id", thread_id, "--json", "--download"],
        timeout=900,
    )


@mcp.tool()
def lovart_upload(file_path: str) -> dict[str, Any]:
    """上传本地图片或视频到 Lovart，供后续编辑任务引用。"""
    return run_lovart(["upload", "--file", file_path], timeout=300)


@mcp.tool()
def lovart_set_generation_mode(mode: Literal["fast", "unlimited"]) -> dict[str, Any]:
    """切换账号生成模式；fast 消耗积分，unlimited 可能排队。"""
    args = ["set-mode", "--fast"] if mode == "fast" else ["set-mode", "--unlimited"]
    return run_lovart(args, timeout=60)


if __name__ == "__main__":
    mcp.run(transport="stdio")
```

---

## 5. 在 Codex 注册 MCP

编辑全局配置文件：

```text
C:\Users\Administrator\.codex\config.toml
```

加入以下配置。若已存在 `[mcp_servers.lovart]`，修改该段即可，**不要创建重复段落**：

```toml
[mcp_servers.lovart]
command = "C:/Users/Administrator/.codex/mcp/lovart/.venv/Scripts/python.exe"
args = ["C:/Users/Administrator/.codex/mcp/lovart/server.py"]
env_vars = ["LOVART_ACCESS_KEY", "LOVART_SECRET_KEY"]
startup_timeout_sec = 15
tool_timeout_sec = 900
default_tools_approval_mode = "prompt"
```

`env_vars` 只转发系统中已有的环境变量；密钥不会写入这个配置文件。

`default_tools_approval_mode = "prompt"` 使 Codex 在调用 MCP 工具前要求批准，特别适用于上传、生成和确认扣费等操作。

---

## 6. 重启与验证

1. 完全退出 Codex Desktop。
2. 重新打开 Codex Desktop。
3. 输入 `/mcp`，确认服务列表出现 `lovart`。
4. 先进行只读测试：

   ```text
   使用 Lovart MCP 列出我的项目。
   ```

5. 再测试生成：

   ```text
   使用 Lovart MCP 生成一张极简风格的咖啡品牌海报。
   ```

若 Lovart 返回 `pending_confirmation`，必须先查看费用并明确确认，才可调用 `lovart_confirm`。

---

## 常见问题

| 现象                       | 处理方式                                                    |
| ------------------------ | ------------------------------------------------------- |
| `未检测到 LOVART_ACCESS_KEY` | 重新执行 `setx`，然后关闭并重新打开 Codex。                            |
| `找不到 Lovart 官方脚本`        | 核对 skill 的实际路径；不要把 `lovart-skill` 仓库移动到其他目录。            |
| `/mcp` 中没有 `lovart`      | 检查 `config.toml` 的路径、TOML 格式和 Python 路径，然后重启 Codex。     |
| 视频任务超时                   | 保持 `tool_timeout_sec = 900`；随后可通过 `lovart_result` 获取结果。 |
| 希望减少重复确认                 | 不建议关闭 `prompt`。生成、上传与付费确认均会改变外部状态。                      |

---

## 不需要做的事

- 不需要再次克隆官方 skill。
- 不需要把 AK/SK 写入 Python 代码或 `config.toml`。
- 不需要从零实现 Lovart 的签名、请求、轮询和下载协议。
- 不应使用未维护的第三方 Lovart Canvas MCP canary 包作为生产方案。
