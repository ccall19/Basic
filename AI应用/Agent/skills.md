# 参考链接
https://agentskills.io/home
https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview

# 1.Overview

A simple, open format for giving agents new capabilities and expertise.

Agent Skills are folders of instructions, scripts, and resources.

Format，一种资源管理的方式：指令（md文档中的正文内容）、脚本、资源


## Why Agent Skills?
用于增强Agent，为其提供 与真实工作相关的 上下文。
- 无需修改Agent本身代码，即可扩展其能力和知识边界
- 将领域知识从Agent逻辑中解耦，便于独立维护和版本管理
- 以文件夹为单位组织，结构直观，易于共享和分发

## What can Agent Skills enable?
- **复用上下文**：将设定好的指令、提示词模板打包，跨项目/跨团队复用
- **注入领域专业知识**：如编码规范、排障流程、架构决策记录等
- **携带可执行脚本**：Skill中可包含脚本，Agent可直接调用执行特定任务
- **附带参考资源**：文档、示例、数据文件等，为Agent提供丰富的 grounding 材料

# 2.What are skills?
一个轻量开源的协议 用于 使用 特定知识 和 工作流 来增强 Agent。

其核心 就是一个文件夹，包含 一个 `SKILL.md` 文件，其中记录Agent的元数据（最少要有 `名字` 和 `描述`）与 instructions 用来告诉 一个 Agent 如何执行一个具体的任务。
`SKILL.md`之外，还可以捆绑scripts、templates和 reference materials。

````sh
my-skill/
├── SKILL.md          # Required: instructions + metadata
├── scripts/          # Optional: executable code
├── references/       # Optional: documentation
└── assets/           # Optional: templates, resources
````

## How skills work？

Skills 通过 **progressive disclosure（渐进式披露）** 来高效管理上下文：

1. **Discovery（发现）**：启动时，agent 仅加载每个 skill 的 `name` 和 `description`（约 100 tokens），只需知道何时可能用到它。
2. **Activation（激活）**：当任务匹配某个 skill 的描述时，agent 才读取完整的 `SKILL.md` 指令到上下文中（建议 < 5000 tokens）。
3. **Execution（执行）**：agent 按指令操作，按需加载 `scripts/`、`references/`、`assets/` 等文件。

这种方式让 agent 保持快速响应，同时在需要时仍能按需获取更多上下文。

## The SKILL.md file
所有 skill 都是开始于 SKILL.md文档 ，包含 yaml 格式的 前言（name | description）和 md 语法格式 的 instructions，一个例子：
````sh
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents.
---

# PDF Processing

## When to use this skill
Use this skill when the user needs to work with PDF files...

## How to extract text
1. Use pdfplumber for text extraction...

## How to fill forms
...
````
### 关于 SKILL.md 更具体一些的说明：
**TOP**
其中 name 和 description 必须放在 md 文档的 顶端
- name: A short identifier
- description: When to use this skill

**BODY**
包含真实的 指示（参看上文案例），对结构和内容没有具体限制。

## Next steps

* [View the specification](https://agentskills.io/specification) to understand the full format.
* [Add skills support to your agent](https://agentskills.io/integrate-skills) to build a compatible client.
* [See example skills](https://github.com/anthropics/skills) on GitHub.
* [Read authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) for writing effective skills.
* [Use the reference library](https://github.com/agentskills/agentskills/tree/main/skills-ref) to validate skills and generate prompt XML.

# 3.Specification
直接参看[`原文档`](https://agentskills.io/specification)

主要介绍了 各个字段 的具体要求，比如 name 字段 要求 1 - 64 个 字符，仅支持小写字母以及 '-' ，同时不要以 '-' 为开始或结尾字符等。

其他不在此赘述。

# 4.Integrate skills into your agent

## Integration Approaches
主要有两种：
**Filesystem-based agents** operate within a computer environment (bash/unix) and represent the most capable option. Skills are activated when models issue shell commands like `cat /path/to/my-skill/SKILL.md`. Bundled resources are accessed through shell commands.

**Tool-based agents** function without a dedicated computer environment. Instead, they implement tools allowing models to trigger skills and access bundled assets. The specific tool implementation is up to the developer.

## Injecting into context
把 skill metadata 嵌入到 system prompt 中，这取决于我们使用的 平台。对于 Claude 系列模型，推荐使用 XML：
````xml
<available_skills>
  <skill>
    <name>pdf-processing</name>
    <description>Extracts text and tables from PDF files, fills forms, merges documents.</description>
    <location>/path/to/skills/pdf-processing/SKILL.md</location>
  </skill>
  <skill>
    <name>data-analysis</name>
    <description>Analyzes datasets, generates charts, and creates summary reports.</description>
    <location>/path/to/skills/data-analysis/SKILL.md</location>
  </skill>
</available_skills>
````
对于基于文件系统的Agent，需要加入一个 location 字段来说明 SKILL.md 的路径。

## Reference implementation
[`skills-ref`](https://github.com/agentskills/agentskills/tree/main/skills-ref)
