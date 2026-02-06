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

## 核心理解 (My Understanding)
*   **运行时结构**：“这个运行时就是 Graph Runtime Agent。”
*   **节点构成**：
    *   **大模型节点**：“负责思考、推理，以及决策下一步的动作。”
    *   **工具节点**：“主要负责执行工具调用。”

## 补充/修正 (Refinements)
*   *(修正注脚：除了模型节点和工具节点，Graph 中还包含 **Hook Nodes (钩子节点)**，用于插入自定义逻辑)*
*   *(注：Agent 在这些节点（Steps）和边（Connections）组成的图中流转)*

## 官方定义与溯源 (Reference)
> [!quote] Source
> 原始文档：[[Agentic-Framework/01-Agents.md]]
>
> > [!info] Original Text
> > Spring AI Alibaba 中的 ReactAgent 基于 Graph 运行时 构建。Graph 由节点（steps）和边（connections）组成... Agent 在这个 Graph 中移动，执行如下节点：Model Node, Tool Node, Hook Nodes。
