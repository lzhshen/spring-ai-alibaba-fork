# 多智能体 (Multi-agent)

Spring AI Alibaba 支持多种 Multi-agent 模式，通过协作解决复杂任务。

## Multi-agent 模式

1. **交接模式 (Handoffs)**: Agent 直接切换控制权。
2. **工具调用模式 (Tool Calling)**: 控制器将其他 Agent 作为工具调用（详情见 [Agent Tool](05-Agent-Tool.md)）。

## 核心参数控制

- **instruction**: 支持占位符（如 `{input}`），实现 Agent 间的数据传递。
- **returnReasoningContent**: 若为 `false`，则不将子 Agent 的中间推理返回给父流程，减少上下文。
- **includeContents**: 控制是否包含父流程的所有历史上下文。
- **outputKey**: 指定输出结果的键名。

## 交接模式 (Handoffs)

在交接模式中，活动 Agent 会变化，直接与用户交互。

### 顺序执行 (Sequential Agent)
Agent 按顺序执行，前者的输出作为后者的输入。

```java
SequentialAgent blogAgent = SequentialAgent.builder()
    .subAgents(List.of(writerAgent, reviewerAgent))
    .build();
```

### 并行执行 (Parallel Agent)
多个 Agent 同时处理相同输入，结果合并。

```java
ParallelAgent parallelAgent = ParallelAgent.builder()
    .subAgents(List.of(proseWriter, poemWriter))
    .mergeStrategy(new CustomMergeStrategy())
    .build();
```

### 路由模式 (LlmRoutingAgent)
使用 LLM 动态决定将请求分发给哪个专家 Agent。

```java
LlmRoutingAgent routingAgent = LlmRoutingAgent.builder()
    .model(chatModel)
    .subAgents(List.of(writerAgent, translatorAgent))
    .build();
```

### 监督者模式 (SupervisorAgent)
LLM 作为监督者，支持子 Agent 执行完后返回监督者，进行多步循环路由。

```java
SupervisorAgent supervisor = SupervisorAgent.builder()
    .model(chatModel)
    .subAgents(List.of(researcher, analyst))
    .build();
```

## 自定义 FlowAgent
通过继承 `FlowAgent` 并实现 `buildSpecificGraph` 方法，可以创建自定义的协作模式。
