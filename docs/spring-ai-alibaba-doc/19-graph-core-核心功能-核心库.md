## 核心库：概念指南

## 图（Graphs）[​](#图graphs "图（Graphs）的直接链接")

Spring AI Alibaba Graph 将智能体工作流建模为图。您可以使用三个关键组件来定义智能体的行为：

1. [State](#state%E7%8A%B6%E6%80%81)：共享的数据结构，表示应用程序的当前快照。它由 `OverAllState` 对象表示。
2. [Nodes](#%E8%8A%82%E7%82%B9nodes)：一个**函数式接口** (`AsyncNodeAction`)，编码智能体的逻辑。它们接收当前的 `State` 作为输入，执行一些计算或副作用，并返回更新后的 `State`。或者使用 `AsyncNodeActionWithConfig`，它可以额外接收 `RunnableConfig` 用于传递上下文。
3. [Edges](#%E8%BE%B9edges)：一个**函数式接口** (`AsyncEdgeAction`)，根据当前的 `State` 确定接下来执行哪个 `Node`。它们可以是条件分支或固定转换。或者使用 `AsyncEdgeActionWithConfig`，它可以额外接收 `RunnableConfig` 用于传递上下文。

通过组合 `Nodes` 和 `Edges`，您可以创建复杂的循环工作流，工作流在工作过程中持续更新 `State`，Spring AI Alibaba 会管理好 `State`，并确保 `State` 在工作流中传递并持久化。

在 Graph 中，`Nodes` 和 `Edges` 就像函数一样 - 它们可以包含 LLM 调用或只是普通的 Java 代码。

简而言之：*节点完成工作，边决定下一步做什么*。

### StateGraph[​](#stategraph "StateGraph的直接链接")

`StateGraph` 类 Spring AI Alibaba Graph 中的核心定义，它通过用户定义的状态策略进行参数化。

### 编译图[​](#编译图 "编译图的直接链接")

要构建您的图，首先定义 [state](#state%E7%8A%B6%E6%80%81)，然后添加 [nodes](#%E8%8A%82%E7%82%B9nodes) 和 [edges](#%E8%BE%B9edges)，最后编译它。编译图是什么意思？为什么需要编译？

编译是一个非常简单的步骤，它提供了对图结构的一些基本检查（没有孤立节点等）
<truncated 19238 bytes>
将作为下一个并行执行。关于并行节点，可参考示例目录中的详细文档描述。

## 会话（Threads）[​](#会话threads "会话（Threads）的直接链接")

会话支持对多个不同运行进行检查点，这对于多租户聊天应用程序和其他需要维护独立状态的场景至关重要。会话是分配给 checkpointer 保存的一系列检查点的唯一 ID。使用 checkpointer 时，必须在运行图时指定 `thread_id`。

```
// 指定会话 ID  
RunnableConfig config = RunnableConfig.builder().threadId("unique-id-1").build();  
  
// 调用 Graph 时传进去  
  
Flux<NodeOutput> stream = graph.stream(Map.of("input", "你好"), config);  
  
//可以在多次调用间传递同一个会话 ID  
RunnableConfig config2 = RunnableConfig.builder().threadId("unique-id-1").build();  
Flux<NodeOutput> stream2 = graph.stream(Map.of("input", "你好"), config2);
```

### Checkpointer（检查点）[​](#checkpointer检查点 "Checkpointer（检查点）的直接链接")

Spring AI Alibaba 具有内置的持久化层，通过 Checkpointers 实现。当您将 checkpointer 与图一起使用时，可以与图的状态进行交互。checkpointer 在每一步保存图状态的\_检查点\_，实现几个强大的功能：

首先，checkpointers 通过允许人类检查、中断和批准步骤来促进**人机协作工作流**。这些工作流需要 checkpointers，因为人类必须能够在任何时间点查看图的状态，并且图必须能够在人类对状态进行任何更新后恢复执行。

其次，它允许在交互之间保持"记忆"。您可以使用 checkpointers 创建会话并在图执行后保存会话的状态。在重复的人类交互（如对话）的情况下，任何后续消息都可以发送到该检查点，它将保留之前的记忆。

每条 Checkpoint 中记录了如下内容，它们可以作为检视和恢复图的基础：

* **state**：这是此时的状态值。
* **nextNodeId**：这是图中接下来要执行的节点的标识符。
