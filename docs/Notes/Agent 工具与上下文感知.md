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

在 Spring AI Alibaba 中，自定义工具不只是单纯的函数，而是具备**上下文感知**能力的组件。

## BiFunction 接口
工具类通常实现 `BiFunction<T, ToolContext, R>` 接口：
*   **输入 1 (T)**：模型传递的业务参数（如查询关键词）。
*   **输入 2 (ToolContext)**：框架运行时注入的上下文（如 UserID, SessionID, TraceID）。
*   **输出 (R)**：返回给模型的执行结果。

这种设计允许工具在执行时利用环境信息，实现"看人下菜碟"或鉴权等高级功能。

## 鲁棒性设计
工具应当捕获内部异常，并返回描述性的错误信息（而非直接抛出 Exception），以便 Agent 能够理解错误原因并尝试自我修正。

## 关联笔记
- [[Agent 定义与 ReAct 范式]]
