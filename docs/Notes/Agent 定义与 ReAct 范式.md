---
tags:
  - Agent
  - ReAct
  - LLM
aliases:
  - ReAct范式
created: 2026-02-05
---

# Agent 定义与 ReAct 范式

## 核心理解 (My Understanding)
*   **Agent 能力**：“Agent 相比普通的大模型调用，增加了以下两个能力：1. 工具调用；2. 根据工具调用的结果进行思考和推理，从而决定下一步采取什么动作。”
*   **ReAct 机制**：“ReAct 的方式能够动态地思考和规划下一步。它将一个目标拆解为多个任务，每次调用完工具后，都会根据返回的结果来判断下一次的执行步骤。”
*   **执行逻辑**：“(a) 如果已经满足条件或达到最大执行次数，则停止；(b) 否则，它会继续执行下一个步骤和动作。”

## 补充/修正 (Refinements)
*   *(注：ReAct 是 Reasoning + Acting 的缩写，核心在于“观察-思考-行动”的闭环)*

## 官方定义与溯源 (Reference)
> [!quote] Source
> 原始文档：[[Agentic-Framework/01-Agents.md]]
>
> > [!info] Original Text
> > 一个 LLM Agent 在循环中通过运行工具来实现目标。Agent 会一直运行直到满足停止条件 —— 即当模型输出最终答案或达到迭代限制时。
> > ReAct（Reasoning + Acting）是一种将推理和行动相结合的 Agent 范式。
