# Memory 短期记忆

Memory（短期记忆）允许 Agent 记住之前的会话内容，从而进行连贯的对话。Spring AI Alibaba 将短期记忆作为 Agent 状态的一部分进行管理，通常存储在 `messages` 键中。

## 核心概念

- **State Management**: 记忆存储在 Graph 的状态中。
- **Checkpointing**: 状态使用 checkpointer 持久化，支持跨请求恢复会话。
- **Thread ID**: 用于区分不同会话的标识符。

## 使用方法

要启用短期记忆，需要在构建 Agent 时配置 `saver`（即 checkpointer）。

### 1. 内存存储（测试用）

```java
import com.alibaba.cloud.ai.graph.checkpoint.savers.MemorySaver;

ReactAgent agent = ReactAgent.builder()
        .saver(new MemorySaver())
        .build();
```

### 2. Redis 存储（生产用）

```java
import com.alibaba.cloud.ai.graph.checkpoint.savers.RedisSaver;

RedisSaver redisSaver = new RedisSaver(redissonClient);
ReactAgent agent = ReactAgent.builder()
        .saver(redisSaver)
        .build();
```

### 3. 使用会话 ID 调用

```java
RunnableConfig config = RunnableConfig.builder()
        .threadId("session_123") // 指定会话 ID
        .build();

// 第一次调用
agent.call("你好，我是 Bob", config);

// 第二次调用（Agent 会记得我是 Bob）
agent.call("我叫什么名字？", config);
```

## 管理上下文长度

随着对话进行，消息历史会变长，可能超过 LLM 的上下文窗口。Spring AI 提供了几种策略来处理这个问题。

### 1. 消息修剪 (Message Trimming)

使用 `MessagesModelHook` 在发送给模型之前修剪消息历史。

```java
@HookPositions({HookPosition.BEFORE_MODEL})
public class MessageTrimmingHook extends MessagesModelHook {
    private static final int MAX_MESSAGES = 10;

    @Override
    public AgentCommand beforeModel(List<Message> previousMessages, RunnableConfig config) {
        if (previousMessages.size() <= MAX_MESSAGES) {
            return new AgentCommand(previousMessages);
        }
        // 保留 SystemMessage + 最后 N 条消息
        // ... 实现修剪逻辑
        return new AgentCommand(trimmedMessages, UpdatePolicy.REPLACE);
    }
}
```

### 2. 消息总结 (Message Summarization)

使用 LLM 定期总结旧消息，用摘要替换详细历史。

```java
// Spring AI 提供了内置的 SummarizationHook（参考 Hooks 章节）
```

### 3. 消息删除

在 `AFTER_MODEL` 钩子中删除不再需要的中间消息或敏感信息。

## 访问和修改记忆

### 在工具中访问

通过 `ToolContext` 访问：

```java
public class UserInfoTool implements BiFunction<String, ToolContext, String> {
    @Override
    public String apply(String input, ToolContext context) {
        RunnableConfig config = (RunnableConfig) context.getContext().get("config");
        String userId = (String) config.metadata("user_id").orElse(null);
        // ...
    }
}
```

### 在 Hooks 中访问

Hooks 可以直接读取和修改 `OverAllState`，从而完全控制记忆。

```java
public class CustomMemoryHook extends ModelHook {
    @Override
    public CompletableFuture<Map<String, Object>> beforeModel(OverAllState state, RunnableConfig config) {
        List<Message> messages = (List<Message>) state.value("messages").get();
        // ...
    }
}
```
