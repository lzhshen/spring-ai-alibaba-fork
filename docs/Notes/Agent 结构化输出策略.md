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

## 核心理解 (My Understanding)
*   **OutputType (静态)**：“提供了一个编译时的类型安全（Type-safe）。”
    *   *(注：通过 Java Class 定义，结构在编译时确定)*
*   **OutputSchema (动态)**：“如果使用 Schema 的话，结构是可以动态生成的。”
    *   *(注：适合运行时才能确定字段的场景)*
*   **流式输出区分**：“通过 output type 来去区分... 总共 6 种类型... 分别对应到这三个节点：模型，工具，Hooks。”
*   **工具完成信号**：“用户能够知道他的这个工具调用返回结束。”
    *   *(注：即 AGENT_TOOL_FINISHED 类型)*

## 补充/修正 (Refinements)
*   *(注：OutputSchema 可以配合 BeanOutputConverter 使用)*

## 官方定义与溯源 (Reference)
> [!quote] Source
> 原始文档：[[Agentic-Framework/01-Agents.md]]
>
> > [!info] Original Text
> > 在某些情况下，你可能希望 Agent 以特定格式返回输出。ReactAgent 提供了两种策略：使用 outputType... 使用 outputSchema。
