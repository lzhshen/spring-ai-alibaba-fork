# 智能体作为工具 (Agent Tool)

在 Multi-agent 系统中，一个 Agent（控制器）可以将另一个 Agent（子智能体）当作工具调用。

## 工作原理

1. 控制器接收输入，根据各工具（子 Agent）的描述决定调用哪个。
2. 子 Agent 作为一个函数执行，返回结果后结束，不与用户直接对话。
3. 控制器接收结果并决定下一步。

## 基本实现

使用 `AgentTool.getFunctionToolCallback(childAgent)` 将 Agent 转化为工具。

```java
ReactAgent writerAgent = ReactAgent.builder()
    .name("writer_agent")
    .description("可以写文章")
    .build();

ReactAgent coordinator = ReactAgent.builder()
    .tools(AgentTool.getFunctionToolCallback(writerAgent))
    .build();
```

## 精细化控制

### 1. 结构化输入 (Input Schema)
使用 `inputSchema` 或 `inputType` 强制要求控制器提供特定的结构化参数。

```java
ReactAgent writerAgent = ReactAgent.builder()
    .inputSchema("""
        { "type": "object", "properties": { "topic": { "type": "string" } } }
    """)
    .build();
```

### 2. 结构化输出 (Output Schema)
使用 `outputSchema` 或 `outputType` 让子 Agent 返回结构化数据，方便控制器解析。

```java
ReactAgent writerAgent = ReactAgent.builder()
    .outputType(ArticleOutput.class)
    .build();
```

## 使用建议

- **描述至上**: 子 Agent 的 `description` 决定了控制器何时会调用它。
- **专注任务**: 工具 Agent 应该是无状态的、专门应对特定子任务的专家。
- **上下文精简**: 如果子 Agent 内部产生大量中间步骤，建议关闭推理内容返回。
