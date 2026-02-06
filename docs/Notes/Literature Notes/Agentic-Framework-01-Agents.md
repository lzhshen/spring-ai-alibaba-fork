## ReactAgent 理论基础

- **ReAct 定义**: ReAct Agent 是相对于 CoT 来说的。它具备三个核心能力 ([[Agentic-Framework/01-Agents.md#什么是 ReAct|来源]])：
    1. **Planning (计划)**: 根据当前的上下文计划，将任务拆分为多个子任务。
    2. **工具调用**: 在执行过程中调用相应的工具。
    3. **感知与决策**: 感知工具调用后的结果，并根据结果决策下一步动作（如判断目标达成或触达限制）。
- **动态性**: 它不是一开始就把所有的任务都规划好，因为最开始掌握的上下文不全。它会根据工具的执行情况、调用情况和返回结果，**动态地规划并调整计划**。 ([[Agentic-Framework/01-Agents.md#ReactAgent 的工作原理|来源]])
- **实现机制**: Spring AI Alibaba 的 Graph 由三类节点组成 ([[Agentic-Framework/01-Agents.md#ReactAgent 的工作原理|来源]])：
    1. **推理节点**: 即大模型的节点。
    2. **工具调用节点**。
    3. **Hooks 节点**。
