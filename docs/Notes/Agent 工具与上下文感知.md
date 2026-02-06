---
tags:
  - Tools
  - Java
  - Context
aliases:
  - BiFunction工具定义
created: 2026-02-05
---

# Agent 工具与上下文感知

## 核心理解 (My Understanding)
*   **接口实现**：“它实现了 apply 接口... 具体是用 BiFunction。”
*   **参数逻辑**：“Function 的输入是由它的参数个数来决定的。”
    *   *(注：因为需要同时传入 **Query** 和 **ToolContext** 两个参数，所以必须使用 BiFunction)*
*   **错误反馈**：“工具抛出异常... 会把错误消息传回给大模型”，从而让 Agent 决定是“修复错误”还是“换别的工具”。

## 补充/修正 (Refinements)
*   *(注：ToolContext 包含了 UserID 等框架注入的元数据，让工具具备上下文感知能力)*

## 官方定义与溯源 (Reference)
> [!quote] Source
> 原始文档：[[Agentic-Framework/01-Agents.md]]
>
> > [!info] Original Text
> > 工具赋予 Agent 执行操作的能力...
> > public class SearchTool implements BiFunction<String, ToolContext, String>
