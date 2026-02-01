# DeepResearch 智能体

DeepResearch 是一个基于 Spring AI Alibaba 构建的深度研究智能体示例，旨在解决复杂的多步骤研究任务。

## 项目简介
普通的智能体往往只能进行简单的工具调用，难以处理深层问题。DeepResearch 通过以下机制实现了“深度”研究：
- **任务规划 (Planning)**: 将大型研究目标拆解为可执行的子任务。
- **子智能体协作 (Sub-agents)**: 不同的 Agent 负责搜索、分析和总结。
- **文件系统访问**: 支持读取和写入研究报告及支撑材料。

## 核心架构
DeepResearch 采用了超越简单工具循环的 Agentic 架构：
1. **Planner**: 负责全局任务规划。
2. **Researcher**: 驱动搜索工具获取外部知识。
3. **Writer**: 负责整合信息并撰写格式美观的报告。

## 技术亮点
- **DAG 编排**: 使用 Spring AI Alibaba Graph 将研究步骤串联。
- **持久化**: 即使研究过程耗时较长，也能通过 Checkpointer 保存进度，支持断点续传。
- **人类介入**: 在生成最终报告前，可以暂停等待人工对研究大纲进行反馈。

## 快速体验
1. 克隆项目并配置 `DASH_SCOPE_API_KEY`。
2. 运行 `DeepResearchApplication`。
3. 输入研究课题（如“2025年AI Agent技术趋势报告”），Agent 将开始自动执行多轮研究。
