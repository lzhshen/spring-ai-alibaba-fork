---
tags:
  - Hooks
  - Agent
  - ControlFlow
aliases:
  - Agent拦截器
  - 钩子节点
created: 2026-02-05
---

# Agent Hook 机制

## 核心理解 (My Understanding)
*   **位置与时机**：“钩子节点可以放在大模型调用之前和调用之后，也可以放在工具的调用前和调用后。”
*   **循环触发**：“对于大模型的调用前后，它好像是每一次调用都会被触发一次。”
    *   *(注：准确来说是 MessagesModelHook 在 ReAct 循环的每次推理前后触发)*
*   **应用场景 (消息修剪)**：“Message Trimming Hooks 能够判断历史消息是否超过了设定的最大限制（Maximum Value）。如果超过了，它就只保留最近的那个 Maximum 消息数，更早之前的消息就会被丢弃掉。”
*   **异常处理 (拦截器)**：“这个拦截器是不是相当于在工具调用完成后，会把错误消息传回给大模型？这样的话，大模型或者 Agent 就会根据工具调用的结果，灵活地决策下一步的动作... 比如：1. 修复错误 2. 换别的工具。”

## 补充/修正 (Refinements)
*   *(注：Interceptor 与 Hook 类似但更底层，ToolErrorInterceptor 专门用于将代码异常转化为 Observation)*

## 官方定义与溯源 (Reference)
> [!quote] Source
> 原始文档：[[Agentic-Framework/01-Agents.md]]
>
> > [!info] Original Text
> > Hooks 允许在 Agent 执行的关键点插入自定义逻辑。
> > MessagesModelHook - 在模型调用前后执行（例如：消息修剪），...每次 reasoning-acting 迭代都会执行。
