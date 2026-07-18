---
title: 使用扣子编程零代码开发“运维巡检 Skill”
created: 2026-07-16
tags:
  - Skills
---
---
# 如何写好`SKILL.md` 

Skills 的开发并不复杂，其核心在于精心编写的 `SKILL.md`，该文件作为核心提示词，指导 LLM 完成特定任务。下面介绍如何撰写高质量的 `SKILL.md`，并探讨如何利用扣子编程加速开发过程。

为了确保 LLM 能够准确地理解和可靠地执行某个 Skill，一份优秀的 `SKILL.md` 应包含以下 5 个关键部分：

- 【定时机】：明确该 Skill 的适用情境或触发条件，即在什么上下文、用户意图或系统状态下，应激活该 Skill。清晰界定使用时机，有助于 LLM 判断何时调用该能力，避免误用或遗漏。
- 【立目标】：用一句简洁、具体的话说明该 Skill 要解决的核心问题。目标描述应聚焦、可衡量，避免模糊或宽泛的表述。例如，“将用户输入的自然语言查询转换为结构化的 SQL 语句”比“帮助用户查询数据”更具指导性。
- 【理规则】：详细说明该 Skill 的执行逻辑与操作步骤，包括输入如何解析、中间如何处理、输出应遵循何种格式等。此部分是该 Skill 实现的操作手册，需条理清晰、逻辑严密，便于 LLM 按步骤推理和执行。
- 【给示例】：提供典型“输入-输出”对作为参考。理想情况下，应同时包含正面示例（展示正确用法）和反面示例（说明常见错误或不适用情形）。示例应贴近真实使用场景，具有代表性，能有效引导模型行为。
- 【划边界】：明确定义该 Skill 的能力边界与限制条件。例如，不处理敏感信息、在数据不足时返回特定提示（如“信息不足，无法生成答案”）、遇到未覆盖的异常情况时采用兜底策略等。这有助于提升系统的鲁棒性与安全性。

基于上述指导原则，开发者既可以手动编写 `SKILL.md`，也可以借助 AI 编程工具自动生成。

---
# Agent Skills 的构建工具

传统的 AI 编程工具（如集成了 Github Copilot、Codex 等 AI 插件的 IDE ）和“扣子”在 Agent Skills 构建领域代表着两种截然不同的技术路径：

- IDE 集成是面向开发者的工作流增强工具；
- 扣子是面向业务角色的低代码/零代码 Agent 工厂。

二者差异的本质源于设计原点的不同：

- IDE 集成从程序员如何更高效编码出发，将 AI 深度嵌入代码编写场景；
- 扣子从非技术用户如何快速组装 Agent 切入，通过可视化编排降低 Agent Skills 构建门槛。

混淆二者定位易导致工具选型失误：试图用 IDE 集成直接构建端到端 Agent Skills、或期望扣子编程支持底层算法定制，均会陷入能力错配。

二者在 Agent Skills 构建中的互补性大于竞争性。典型的协作模式是：

- 开发者用 IDE 集成快速实现定制化技能，然后将该技能封装为标准 API 并注册至企业技能库；
- 业务人员再通过扣子等平台将此技能与预置 Agent Skills 组合，形成面向具体场景的数字员工。

举一个具体的例子：

- 开发者用 IDE 集成编写对接企业私有 ERP 系统的订单状态查询技能，并以 FastAPI 方式封装；
- 业务人员在 Codex 中将此订单状态查询技能与自动回复客户消息技能组合成为订单查询小助手。

---
# 基于扣子编程开发 Agent Skills

“扣子编程”平台提供了完整的基于自然语言编程的 Skills 开发支持，极大地简化了开发者从构思到实现的全过程。

让我们开始吧：

1. 打开“扣子编程”，进入其主页。
2. 在主页点击“技能”标签，即可进入 Skills 开发界面。
3. 随后，在对话框中输入如下提示词：

```plain
帮我编写一个“运维巡检”，可以自动进行服务器的巡检（检查 CPU、内存、磁盘、Docker 容器状态），并输出一份巡检报告。

为了确保 LLM 能够准确地理解和可靠地执行某个 Skill，一份优秀的 SKILL.md 应包含以下 5 个关键部分：

1. 定时机。明确该 Skill 的适用情境或触发条件，即在什么上下文、用户意图或系统状态下，应激活该 Skill。清晰界定使用时机，有助于模型判断何时调用该能力，避免误用或遗漏。
2. 立目标。用一句简洁、具体的话说明该 Skill 要解决的核心问题。目标描述应聚焦、可衡量，避免模糊或宽泛的表述。例如，“将用户输入的自然语言查询转换为结构化的 SQL 语句”比“帮助用户查询数据”更具指导性。
3. 理规则。详细说明该 Skill 的执行逻辑与操作步骤，包括输入如何解析、中间如何处理、输出应遵循何种格式等。此部分是该 Skill 实现的操作手册，需条理清晰、逻辑严密，便于模型按步骤推理和执行。
4. 给示例。提供典型输入-输出对作为参考。理想情况下，应同时包含正面示例（展示正确用法）和反面示例（说明常见错误或不适用情形）。示例应贴近真实使用场景，具有代表性，能有效引导模型行为。
5. 划边界。明确定义该 Skill 的能力边界与限制条件。例如，不处理敏感信息、在数据不足时返回特定提示（如“信息不足，无法生成答案”）、遇到未覆盖的异常情况时采用兜底策略等。这有助于提升系统的鲁棒性与安全性。
```

该提示词既明确了功能需求，又嵌入了前文所述的 `SKILL.md` 写作规范，有助于引导 AI 生成高质量、结构完整的 Skills。

4. 提交后，扣子编程将跳转至开发与调试界面：
	- 左侧展示 AI 的开发过程（类似于使用 IDE 集成的体验）；
	- 右侧为沙盒环境，支持直接调用名为扣子产品的通用 Agent 对 Skills 进行测试。

5. 在开发完成后，点击左侧开发区域右上角的文件夹图标，将弹出文件目录视图。该视图完整展示了生成的 Skill 结构，包括 `SKILL.md`、`scripts/` 和 `references/` 等标准组件。

6. 点击任意文件即可预览其内容。例如，`SKILL.md` 的部分内容结构严谨，清晰定义了任务目标、前置条件、执行流程及输出格式。
   
从这个例子可以看出，扣子编程采用了如下设计思路：

- 使用 Python 的 `psutil` 库采集系统指标；
- 将采集逻辑封装在 `scripts/collect_system_info.py` 中；
- 依据 `references/inspection-standards.md` 中定义的健康阈值进行评估；
- 最终整合结果，生成标准化巡检报告。

以下是 `collect_system_info.py` 中获取 CPU 指标的代码片段：

```python
from typing import Dict, Any
import psutil


def get_cpu_info() -> Dict[str, Any]:
    try:
        cpu_percent = psutil.cpu_percent(interval=1)
        cpu_count = psutil.cpu_count(logical=True)
        cpu_count_physical = psutil.cpu_count(logical=False)
        load_avg = psutil.getloadavg() if hasattr(psutil, "getloadavg") else [0, 0, 0]

        return {
            "success": True,
            "percent": round(cpu_percent, 2),
            "count": cpu_count,
            "physical_core_count": cpu_count_physical,
            "load_avg_1m": round(load_avg[0], 2),
            "load_avg_5m": round(load_avg[1], 2),
            "load_avg_15m": round(load_avg[2], 2),
        }
    except Exception as e:
        return {
            "success": False,
            "error": str(e),
        }
```

这段代码充分利用了 `psutil` 提供的系统接口：

> - 第 8 行获取 CPU 使用率；
> - 第 9、10 行分别获取逻辑与物理核心数；
> - 第 11 行读取系统负载均值；
> - 最终，所有指标被封装为结构化字典返回，便于后续处理与判断。

此外，`references/inspection-standards.md` 定义了各项指标的健康阈值标准。每个标准均标注了数据来源，确保评估逻辑有据可依，避免因指标缺失导致推理失败。

7. 最后点击“下载”按钮，即可将整个 Skill 文件夹下载下来，以便将其复用到其他支持 Skills 的 Agent 平台（如 Cherry Studio、Codex、Claude Code、OpenClaw 等）。

---
# 基于 Cherry Studio 测试刚刚编写好的“运维巡检 Skill”

Skills 本质上是为 Agent 提供的专家经验包，因此必须在支持 Skills 加载机制的 Agent 环境中进行测试。下面我们选用熟悉的 Agent 产品 Cherry Studio 作为测试平台。除此以外，也可使用 Claude Code、Open Code、Codex、Open Claw、Hermes 等其他支持 Skill 的 Agent 进行类似验证。

根据 `SKILL.md` 中的依赖说明，“运维巡检 Skill”要求系统满足以下条件。

- Python 版本 >= 3.7。
- 安装 `psutil` 库，版本 >= 5.9.0。

通过如下命令可安装所需的依赖。

```bash
pip install "psutil>=5.9.0"
```

依赖安装就绪后，将生成的 Skill 文件夹（`ops-inspection`）拷贝至 Cherry Studio 的 Agent 所在工作目录中的`.claude`下的 Skill 存放目录。

```text
/你所指定的Agent工作目录/.claude/skill
```

在 Cherry Studio 中向 Agent 发起对话，首先发送消息：

```text
你是否存在 ops-inspection 这个 Skill
```

用于检测 Skill 是否被 Agent 加载。系统返回结果表明，该 Skill 已被成功加载。

随后，发送新的对话：

```text
使用该 Skill 执行一次巡检
```

这个对话用于测试 Skill 的执行效果。最终输出为服务器巡检报告，包含总体评估、CPU、内存、磁盘、Docker 等状态，以及对应的优先级建议。

从结果可见，报告内容真实、格式规范、判断依据明确，完全达到了预设的巡检目标。

整个过程无须编写任何运维脚本，无须配置告警推送逻辑，也无须手动调度任务。仅通过几轮自然语言对话，便完成了传统运维中通常需要数小时编码、测试与部署的工作。这不仅大幅降低了自动化门槛，也充分体现了 Skills + Agent 架构在实际工程场景中的高效性与实用性。

---