# Core Concepts (核心概念)

Spring AI Alibaba Graph Core 基于图论模型构建 AI 工作流，核心包含以下三个要素：

1.  **State (状态)**: 定义图中的数据模型。`OverAllState` 保存全局共享数据。
2.  **Nodes (节点)**: 执行具体的工作逻辑。节点完成工作后，返回更新的状态。
3.  **Edges (边)**: 决定流程的流转方向。
    - **Normal Edges**: 顺序流转。
    - **Conditional Edges**: 基于逻辑判断决定下一个节点。

## StateGraph and Compilation (状态图与编译)

`StateGraph` 类是框架的核心定义。开发者必须运行 `.compile()` 来对图结构进行基本检查并配置运行时参数（如 checkpoints）。

```java
// Compile the graph
CompiledGraph graph = stateGraph.compile();
```

## OverAllState and KeyStrategy (全局状态与键策略)

`KeyStrategy` 管理节点如何更新状态键。

-   **ReplaceStrategy**: 用新值完全替换旧值。
-   **AppendStrategy**: 将新值追加到旧值中。
-   **RemoveByHash**: 从 `AppendStrategy` 列表中删除项。

```java
keyStrategyMap.put("messages", new AppendStrategy());
// Deletion example
Map.of("messages", RemoveByHash.of("message2.1"))
```

## Serialization (序列化)

系统默认使用 Jackson 进行状态克隆和跨执行持久化状态。用户可以定制 StateGraph 中的默认 Serializer 或提供 `CustomizedSerializer`。

```java
// Custom serializer implementation
class CustomizedSerializer extends SpringAIJacksonStateSerializer {
    public CustomizedSerializer() {
        super(OverAllState::new);
        var module = new SimpleModule();
        objectMapper.registerModule(module);
    }
}
```

## Nodes and Edges (节点与边)

"START" 和 "END" 代表入口和出口点。

-   **Normal Edges**: 直接连接节点。
-   **Conditional Edges**: 使用路由函数决定下一条路径。

```java
graph.addConditionalEdges("nodeA", edge_async(state -> "nodeB"),
      Map.of("nodeB", "nodeB", "nodeC", "nodeC"));
```

## Threads and Checkpointers (线程与检查点)

会话利用 `thread_id` 来维护“记忆”并支持“人机协作工作流”。Checkpointers 捕获当前的“state”和“nextNodeId”以进行持久化。

```java
RunnableConfig config = RunnableConfig.builder().threadId("unique-id-1").build();
graph.stream(Map.of("input", "你好"), config);
```
