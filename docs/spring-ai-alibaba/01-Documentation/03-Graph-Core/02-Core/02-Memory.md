# 内存管理 (Memory Management)

Spring AI Alibaba Graph 提供了一套完整的内存管理机制，支持短期（会话级）和长期（持久化）内存，使智能体能够维持多轮对话和跨会话的上下文。

## 短期内存 (Short-term Memory)

短期内存指单个会话 (threadId) 内的上下文。通过 `MemorySaver`（检查点器）实现。

### 实现多轮对话
你需要定义一个消息追加策略 (`AppendStrategy`)，并配置 `Saver`。

```java
// 1. 定义状态合并策略 (消息列表使用追加模式)
KeyStrategyFactory strategyFactory = () -> {
    Map<String, KeyStrategy> strategies = new HashMap<>();
    strategies.put("messages", new AppendStrategy());
    return strategies;
};

// 2. 配置内存存储
MemorySaver memory = new MemorySaver();
SaverConfig saverConfig = SaverConfig.builder().register(memory).build();

// 3. 编译图
CompiledGraph graph = workflow.compile(
    CompileConfig.builder().saverConfig(saverConfig).build()
);

// 4. 使用 threadId 调用
RunnableConfig config = RunnableConfig.builder().threadId("session-123").build();
graph.invoke(input, config);
```

## 长期内存 (Long-term Memory)

长期内存用于跨对话存储用户画像、偏好等。主要通过 `Store` 组件实现。

### 核心概念
- **Store**: 抽象接口，常见实现有 `MemoryStore`, `RedisStore`。
- **Namespace**: 数据的逻辑分类（如 `user_profiles`, `knowledge_base`）。
- **Key**: 具体条目的唯一标识。

### 存储数据
```java
Store store = new MemoryStore();
store.putItem(StoreItem.of(
    List.of("users", "preferences"), 
    "color", 
    Map.of("value", "blue")
));
```

## 短期 vs 长期内存

| 特性 | 短期内存 (Checkpointer) | 长期内存 (Store) |
| :--- | :--- | :--- |
| **持久化粒度** | 会话级 (ThreadId) | 全局/用户级 (Namespace) |
| **主要用途** | 维持对话连贯、重放执行 | 个性化偏好、知识积累 |
| **实现类** | `MemorySaver`, `RedisSaver` | `MemoryStore`, `RedisStore` |
| **访问方式** | 自动加载 (RunnableConfig) | 显式读写 (Tools/Hooks) |

## 最佳实践
1. **自动提取**: 在 `afterModel` Hook 中分析对话，将有价值的信息存入 `Store`。
2. **上下文注入**: 在 `beforeModel` Hook 中根据 `userId` 从 `Store` 加载偏好，注入到 System Prompt 增强表现。
