---
title: 导入 Skill 的方法
created: 2026-08-03
source: Cherry Studio
tags:
  - Skills
  - Agent_Skills
  - Import_AgentSkills
---
---

# 导入 Skill

如果你已经在使⽤ Claude Code，很可能已经积累了一些好用的 Skills。
好消息是，这些 Skills 可以直接导入 Antigravity 使用，因为两者采用相同的 Skill 标准格式。

## Claude Code 的 Skills 路径

Claude Code 的 Skills 储存在以下位置：

| 类型 | 路径 |
| :--- | :--- |
| 全局 Skills | `~/.claude/skills/` |
| 项目 Skills | `<project>/.claude/skills/` |

## 导入方式一：复制整个文件夹

最简单的方式是直接复制 Skill 文件夹：

```bash
# 将 Claude Code 的全局 Skill 复制到 Antigravity
cp -r ~/.claude/skills/my-skill ~/.gemini/antigravity/skills/

# 或复制到工作区
cp -r ~/.claude/skills/my-skill ./.agent/skills/
```

## 导入方式二：建立符号链接

如果你想同时在两个工具中使用同一个 Skill，可以建立符号链接：

```bash
# 在 Antigravity 全局目录建立链接
ln -s ~/.claude/skills/my-skill ~/.gemini/antigravity/skills/my-skill

# 或在工作区建立链接
ln -s ~/.claude/skills/my-skill ./.agent/skills/my-skill
```

这样修改一处，两边都会同步更新。

## 导入方式三：批量导入所有 Skills

如果你有多个 Skills 想一次导入：

```bash
# 批量复制所有 Claude Code Skills
for skill in ~/.claude/skills/*/; do
  skill_name=$(basename "$skill")
  cp -r "$skill" "~/.gemini/antigravity/skills/$skill_name"
done

# 或批量建立符号链接
for skill in ~/.claude/skills/*/; do
  skill_name=$(basename "$skill")
  ln -s "$skill" "~/.gemini/antigravity/skills/$skill_name"
done
```

---
# 验证导入成功

## 在 Agent 交互界面验证

导入后，可以在 Antigravity 中验证 Skill 是否正确载入：

1. 开启一个新的 Agent 对话
2. 询问 Agent：「列出所有可用的 Skills」
3. 确认你导入的 Skill 出现在列表中

如果 Skill 没有出现，检查：

* 文件夹结构是否正确
* `SKILL.md` 是否存在且格式正确
* 符号链接是否有效（使用 `ls -la` 确认）

## 兼容性注意事项

大多数 Claude Code Skills 可以直接在 Antigravity 中使用，但有几点需要注意：

| 项目              | 说明                                         |
| :-------------- | :----------------------------------------- |
| **SKILL.md 格式** | 完全兼容，无需修改                                  |
| **脚本路径**        | 如果脚本中使用了硬编码路径（如 `~/.claude/`），需要调整         |
| **MCP 工具**      | Claude Code 特有的 MCP 工具在 Antigravity 中可能不可用 |
| **环境变量**        | 确认 Skill 使用的环境变量在 Antigravity 环境中有设定       |

## 调整路径的技巧

如果 Skill 的脚本中有 Claude Code 特定的路径，可以使用环境变量让它更通用：

```python
import os

# 取代硬编码路径
# 旧: skill_dir = os.path.expanduser("~/.claude/skills/my-skill")
# 新:
skill_dir = os.environ.get("SKILL_DIR", os.path.dirname(__file__))
```

或在 `SKILL.md` 中使用相对路径：

````markdown
## 使用方式

执行脚本时使用相对于 Skill 目录的路径：

```bash
python ./scripts/my_script.py --help
```
````

---


