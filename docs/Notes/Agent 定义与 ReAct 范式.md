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

**Agent** 是将大语言模型（LLM）与外部工具（Tools）结合的自动化系统。相比于单纯的 LLM 调用，Agent 增加了 **自主规划（Reasoning）** 和 **工具使用（Acting）** 两个核心能力。

## ReAct 循环
Agent 的运行基于 **ReAct (Reasoning + Acting)** 范式，形成一个闭环：
1.  **思考 (Reasoning)**：模型分析当前情况，决定下一步行动。
2.  **行动 (Acting)**：执行工具调用。
3.  **观察 (Observation)**：接收工具执行的结果反馈。
4.  **循环**：基于观察结果再次思考，直到任务完成或达到停止条件。

此范式让 Agent 能够处理复杂任务，并在不确定的环境中动态调整策略。

## 关联笔记
- [[Spring AI Alibaba Graph 运行时架构]] - 实现 ReAct 的底层架构
