# 记忆管理（Memory）高级指南

Spring AI Alibaba 提供了灵活的记忆管理机制，支持短期记忆（对话上下文中）和长期记忆（持久化存储）。

## 记忆类型

1. **短期记忆 (Short-term Memory)**:
    - 基于 `MemorySaver`（如 `MemorySaver`, `PostgreSQLSaver`, `RedisSaver`）。
    - 存储当前的对话历史和执行状态。
    - 通过 `threadId` 隔离。

2. **长期记忆 (Long-term Memory)**:
    - 基于 `MemoryStore`。
    - 存储用户画像、偏好、知识等。
    - 通过 `namespace` 和 `key` 组织。

## 存储结构

长期记忆以 JSON 形式存储。

```java
MemoryStore store = new MemoryStore();
List<String> namespace = List.of("user_123", "profiles");

// 存入记忆
Map<String, Object> data = Map.of("color_preference", "blue");
store.putItem(StoreItem.of(namespace, "preferences", data));

// 获取记忆
Optional<StoreItem> item = store.getItem(namespace, "preferences");
```

## 自动管理记忆 (ModelHook)

通过 `ModelHook`（特别是 `beforeModel`），可以在模型调用前自动注入长期记忆到 System Prompt 中。

```java
ModelHook memoryHook = new ModelHook() {
    @Override
    public CompletableFuture<Map<String, Object>> beforeModel(OverAllState state, RunnableConfig config) {
        String userId = (String) config.metadata("user_id").get();
        // 从 Store 加载用户信息
        Store store = config.store();
        Optional<StoreItem> profile = store.getItem(List.of("user_profiles"), userId);
        
        // 注入到消息列表中
        String context = "用户信息: " + profile.get().getValue();
        // ...逻辑：更新或添加 SystemMessage
        return CompletableFuture.completedFuture(Map.of("messages", newMessages));
    }
};
```

## 显式记忆操作 (Tools)

你可以向 Agent 提供专门的工具来读写长期记忆。

- `saveUserInfo`: Agent 学习到新信息时调用。
- `getUserInfo`: Agent 需要了解背景时调用。

## 跨会话记忆

由于长期记忆存储在 `Store` 中，不同的 `threadId`（代表不同会话）只要访问同一个 `Store` 并使用正确的 `namespace/key`，就可以实现跨会话的记忆共享。

## 最佳实践

- **结构化存储**: 存储原始 JSON 数据，而不是格式化好的文本，以便不同节点按需转换。
- **命名空间隔离**: 使用 `userId` 或 `orgId` 作为 `namespace` 的一部分。
- **结合使用**: 短期记忆用于维持对话连贯性，长期记忆用于个性化体验。
- **学习机制**: 在 `afterModel` 中分析对话，自动提取并更新用户偏好。
