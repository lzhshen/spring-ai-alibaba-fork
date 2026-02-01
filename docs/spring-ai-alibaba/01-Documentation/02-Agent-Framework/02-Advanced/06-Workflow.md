# 工作流 (Workflow) 编排

Spring AI Alibaba Graph 是底层核心引擎，负责将编排逻辑转化为图（DAG）。

## 核心概念

- **State (状态)**: `Map<String, Object>`，在节点间传递数据的载体。
- **Node (节点)**: 执行单元（LLM、自定义代码、Agent）。
- **Edge (边)**: 控制流，定义下一步去哪。

## 定义自定义节点 (Node)

实现 `NodeAction` 或 `NodeActionWithConfig` 接口。

```java
public class MyNode implements NodeAction {
    @Override
    public Map<String, Object> apply(OverAllState state) throws Exception {
        String input = (String) state.value("input").get();
        // 核心逻辑...
        return Map.of("result", "processed data");
    }
}
```

## Agent 作为节点

通过 `asNode()` 方法，`ReactAgent` 可以无缝集成到工作流中。

```java
workflow.addNode(agent.name(), agent.asNode(
    true, // includeContents: 包含父图上下文
    false // returnReasoningContents: 是否返回推理详情
));
```

## 构建与运行

1. **组合容器**: `StateGraph`。
2. **定义控制边**: `addEdge`, `addConditionalEdges`。
3. **编译执行**: `compile()` 后使用 `invoke()` 或 `stream()`。

```java
CompiledGraph graph = workflow.compile();

// 流式执行获取实时进度
graph.stream(Map.of("input", "..."))
    .doOnNext(output -> System.out.println(output.node()))
    .blockLast();
```

## 运行时特性

- **Streaming**: 原生支持每个节点的增量输出。
- **Human In the Loop**: 允许中途暂停等待审批。
- **Memory**: 节点间和跨会话的状态管理。
