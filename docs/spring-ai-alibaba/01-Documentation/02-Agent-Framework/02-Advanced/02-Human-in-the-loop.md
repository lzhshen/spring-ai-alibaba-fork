# 人工介入（Human-in-the-Loop）

人工介入（HITL）是构建安全可靠 Agent 应用的关键。它允许在 Agent 执行敏感操作（如文件写入、数据库修改等）之前通过人工审批。

## 核心概念

HITL 在 Spring AI Alibaba 中主要通过 `HumanInTheLoopHook` 实现。它允许：
- **中断决策 (Interruption Decision)**: 暂停执行并等待人工决策。
- **决策类型**:
    - `approve`: 批准并继续执行。
    - `edit`: 修改参数后继续执行。
    - `reject`: 拒绝操作并提供反馈。

## 配置中断

在创建 Agent 时，通过 `HumanInTheLoopHook` 指定哪些工具需要审批。

```java
// 创建人工介入Hook
HumanInTheLoopHook humanInTheLoopHook = HumanInTheLoopHook.builder()
    .approvalOn("write_file", ToolConfig.builder()
        .description("文件写入操作需要审批")
        .build())
    .approvalOn("execute_sql", ToolConfig.builder()
        .description("SQL执行操作需要审批")
        .build())
    .build();

// 创建Agent并注入Hook
ReactAgent agent = ReactAgent.builder()
    .name("approval_agent")
    .hooks(List.of(humanInTheLoopHook))
    .saver(new MemorySaver()) // 必须配置检查点
    .build();
```

> [!IMPORTANT]
> 人工介入必须配置**检查点保存器 (Checkpoint Saver)**，因为中断需要跨请求持久化执行状态。

## 响应中断

当工具调用触发中断时，`agent.invokeAndGetOutput()` 会返回 `InterruptionMetadata`。

```java
Optional<NodeOutput> result = agent.invokeAndGetOutput("删除过期记录", config);

if (result.isPresent() && result.get() instanceof InterruptionMetadata) {
    InterruptionMetadata interruption = (InterruptionMetadata) result.get();
    // 展示给用户进行审核
    List<InterruptionMetadata.ToolFeedback> toolFeedbacks = interruption.toolFeedbacks();
}
```

## 恢复执行

用户提供反馈后，使用相同的 `threadId` 和包含反馈的 `RunnableConfig` 恢复执行。

```java
// 构建批准反馈
InterruptionMetadata approvalMetadata = feedbackBuilder.build();

RunnableConfig resumeConfig = RunnableConfig.builder()
    .threadId(threadId)
    .addMetadata(RunnableConfig.HUMAN_FEEDBACK_METADATA_KEY, approvalMetadata)
    .build();

// 恢复执行（输入内容为空）
agent.invokeAndGetOutput("", resumeConfig);
```

## 在 Workflow 中使用

在 `StateGraph` 工作流中嵌套 Agent 时，HITL 同样适用。
- 工作流会在 Agent 节点触发中断时自动暂停。
- 恢复时需通过 `compiledGraph.invokeAndGetOutput()` 处理。

## 最佳实践

1. **始终使用检查点**: 确保状态不会丢失。
2. **清晰的描述**: 在 `ToolConfig` 中解释为什么要审批。
3. **超时处理**: 考虑人工长时间不响应的情况。
4. **一致性**: 恢复时使用相同的 `threadId`。
