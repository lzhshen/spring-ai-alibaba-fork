# Graph 工作流快速入门

Spring AI Alibaba Graph 是构建复杂智能体工作流的核心引擎。它将任务分解为**节点 (Node)**，通过**边 (Edge)** 连接，并由共享的**状态 (State)** 传递上下文。

## 核心概念

1. **状态 (State)**: 图中所有节点共享的记忆载体，具体表现为一个 `Map<String, Object>`。
2. **节点 (Node)**: 独立的逻辑单元。它接收当前状态，执行操作（如调用 LLM），并返回状态增量。
3. **边 (Edge)**: 定义节点间的流转规则。
    - **普通边 (`addEdge`)**: 直接连接两个节点。
    - **条件边 (`addConditionalEdges`)**: 根据状态动态转向。

## 构建流程

### 1. 实现节点
通过 `node_async` 或 `node` 包装逻辑。

```java
public class MyNode implements NodeAction {
    @Override
    public Map<String, Object> apply(OverAllState state) {
        // ...逻辑
        return Map.of("result", "data");
    }
}
```

### 2. 组装图

```java
StateGraph workflow = new StateGraph(keyStrategyFactory)
    .addNode("start_node", node_async(new StartNode()))
    .addNode("end_node", node_async(new EndNode()))
    .addEdge(START, "start_node")
    .addEdge("start_node", "end_node")
    .addEdge("end_node", END);
```

### 3. 配置持久化与中断 (Human-in-the-loop)
使用 `MemorySaver` 保存检查点，并设置中断点。

```java
var compileConfig = CompileConfig.builder()
    .saverConfig(SaverConfig.builder().register(new MemorySaver()).build())
    .interruptBefore("human_review") // 在审核前暂停
    .build();

CompiledGraph app = workflow.compile(compileConfig);
```

## 运行工作流

使用 `stream()` 获取每个节点的输出流。

```java
Flux<NodeOutput> stream = app.stream(initialState, config);
stream.doOnNext(output -> log.info("Node: {}", output)).blockLast();
```

## 恢复执行 (Resume)
当工作流在中断点暂停后，可以使用 `updateState` 提供反馈并继续。

```java
// 更新状态（如注入人工审批结果）
var updatedConfig = app.updateState(config, Map.of("approved", true), null);

// 传入 null 以继续之前保存的进度
app.stream(null, updatedConfig).blockLast();
```
