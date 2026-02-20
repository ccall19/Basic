# Overview

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