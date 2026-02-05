---
tags:
  - Output
  - StructuredData
  - Java
aliases:
  - 结构化输出
created: 2026-02-05
---

# Agent 结构化输出策略

Agent 输出结构化数据（如 JSON）主要有两种策略，适用于不同场景。

## 1. OutputType (静态/强类型)
*   **机制**：直接绑定一个 Java POJO 类（如 `PoemOutput.class`）。
*   **特点**：**编译时静态定义**。
*   **优势**：类型安全，IDE 自动补全，编译器检查。
*   **适用**：业务结构固定，明确知道返回字段的场景。

## 2. OutputSchema (动态/灵活)
*   **机制**：传入一个描述 JSON 结构的字符串 Schema。
*   **特点**：**运行时动态生成**。
*   **优势**：极其灵活，可以在运行时根据用户意图构建不同的输出结构（如动态表单、通用网关）。
*   **适用**：低代码平台、结构不确定的动态场景。

## 流式输出类型
通过 `OutputType` 枚举区分流式消息：
*   `AGENT_MODEL_STREAMING`：模型生成的文本（包含 Thinking 过程）。
*   `AGENT_TOOL_FINISHED`：工具执行完成的信号（用于前端展示进度）。

## 关联笔记
- [[Agent 定义与 ReAct 范式]]
