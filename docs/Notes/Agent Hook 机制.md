---
tags:
  - Hooks
  - Agent
  - ControlFlow
aliases:
  - Agent拦截器
created: 2026-02-05
---

# Agent Hook 机制

**Hook (钩子)** 是介入 Agent 自动化流程的控制点，类似于流水线上的质检员或调节阀。

## 常见类型与位置
1.  **AgentHook**：
    *   位置：`BEFORE_AGENT` / `AFTER_AGENT`
    *   作用：在 Agent 整体执行的开始或结束时触发（只执行一次）。
2.  **MessagesModelHook**：
    *   位置：`BEFORE_MODEL` / `AFTER_MODEL`
    *   作用：在 ReAct 循环的每一次模型调用前后触发。
    *   典型应用：**Context Window 管理**（如 `MessageTrimmingHook`），防止历史消息过长超出 Token 限制。

## 拦截器 (Interceptor)
比 Hook 更底层的控制机制，可拦截 `ModelRequest` 或 `ToolCallRequest`。
*   **ToolErrorInterceptor**：将工具执行异常转化为自然语言的 Observation，防止 Agent 崩溃，引导模型重试。

## 关联笔记
- [[Spring AI Alibaba Graph 运行时架构]]
