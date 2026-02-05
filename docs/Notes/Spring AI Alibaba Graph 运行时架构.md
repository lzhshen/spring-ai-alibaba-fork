---
tags:
  - SpringAIAlibaba
  - Architecture
  - Graph
aliases:
  - Agent Graph Runtime
created: 2026-02-05
---

# Spring AI Alibaba Graph 运行时架构

Spring AI Alibaba 的 `ReactAgent` 基于 **Graph Runtime** 构建。Agent 的执行流程并非线性脚本，而是在一个由 **节点 (Nodes)** 和 **边 (Edges)** 组成的图结构中流转。

## 核心节点类型
1.  **Model Node (模型节点)**：负责调用 LLM 进行推理、决策和生成回复。
2.  **Tool Node (工具节点)**：负责执行具体的工具逻辑。
3.  **Hook Node (钩子节点)**：在流程的关键位置（如模型调用前后）插入自定义逻辑。

这种架构使得 Agent 的行为具备高度的可编排性和扩展性。

## 关联笔记
- [[Agent 定义与 ReAct 范式]]
- [[Agent Hook 机制]]
