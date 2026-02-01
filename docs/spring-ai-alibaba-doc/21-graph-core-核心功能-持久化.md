## 持久化

Spring AI Alibaba Graph 具有内置的持久化层，通过检查点（Checkpointers）实现。当您使用检查点编译图时，检查点会在每个超级步骤（super-step）保存图状态的`检查点`。这些检查点保存到一个`会话`（thread）中，可以在图执行后访问。

由于`会话`允许在执行后访问图的状态，因此几个强大的功能都成为可能，包括人在回路中（human-in-the-loop）、内存、时间旅行和容错能力。下面，我们将详细讨论这些概念。

## 会话[​](#会话 "会话的直接链接")

会话是分配给检查点器保存的每个检查点的唯一 ID 或会话标识符。它包含一系列运行的累积状态。当执行运行时，图的底层状态将被持久化到会话。

当使用检查点调用图时，您**必须**在配置的 `RunnableConfig` 中指定一个 `threadId`。

```
RunnableConfig config = RunnableConfig.builder()  
  .threadId("1")  
  .build();
```

可以检索会话的当前和历史状态。要持久化状态，必须在执行运行之前创建会话。

## 检查点（Checkpoints）[​](#检查点checkpoints "检查点（Checkpoints）的直接链接")

会话在特定时间点的状态称为检查点。检查点是在每个超级步骤保存的图状态快照，由 `StateSnapshot` 对象表示，具有以下关键属性：

* `config`: 与此检查点关联的配置。
* `metadata`: 与此检查点关联的元数据。
* `values`: 此时状态通道的值。
* `next`: 图中下一个要执行的节点名称元组。
* `tasks`: 包含有关下一个要执行的任务的信息的 `PregelTask` 对象元组。

检查点是持久化的，可以用于在稍后的时间恢复会话的状态。

让我们看看当一个简单的图被调用时保存了哪些检查点：

```
import com.alibaba.cloud.ai.graph.CompileConfig;  
import com.alibaba.cloud.ai.graph.CompiledGraph;  
import com.alibaba.cloud.ai.graph.Ke
<truncated 9309 bytes>
til.Map;  
  
KeyStrategyFactory keyStrategyFactory = () -> {  
  Map<String, KeyStrategy> keyStrategyMap = new HashMap<>();  
  keyStrategyMap.put("foo", new ReplaceStrategy());  // 替换策略  
  keyStrategyMap.put("bar", new AppendStrategy());   // 追加策略  
  return keyStrategyMap;  
};  
  
RunnableConfig config = RunnableConfig.builder()  
      .threadId("1")  
      .build();  
  
Map<String, Object> updates = new HashMap<>();  
updates.put("foo", 2);  
updates.put("bar", List.of("b"));  
  
graph.updateState(config, updates, null);  
System.out.println("State updated successfully");
```

那么图的新状态将是：

```
{"foo": 2, "bar": ["a", "b"]}
```

`foo` 键（通道）被完全更改（因为该通道没有指定归约器，所以 `updateState` 覆盖它）。但是，为 `bar` 键指定了归约器（AppendStrategy），因此它将 `"b"` 追加到 `bar` 的状态。

## 检查点器实现[​](#检查点器实现 "检查点器实现的直接链接")

Spring AI Alibaba 提供了多种检查点器实现：

### MemorySaver[​](#memorysaver "MemorySaver的直接链接")

内存检查点器，将检查点保存在内存中：

```
import com.alibaba.cloud.ai.graph.checkpoint.savers.MemorySaver;  
import com.alibaba.cloud.ai.graph.checkpoint.config.SaverConfig;  
import com.alibaba.cloud.ai.graph.checkpoint.constant.SaverConstant;  
  
SaverConfig saverConfig = SaverConfig.builder()  
  .register(SaverConstant.MEMORY, new MemorySaver())  
  .build();
```

### PostgreSqlSaver[​](#postgresqlsaver "PostgreSqlSaver的直接链接")

PostgreSQL 数据库检查点器，详见 [PostgreSQL 检查点持久化](/docs/frameworks/graph-core/examples/checkpoint-redis)。

### RedisSaver[​](#redissaver "RedisSaver的直接链接")

Redis 检查点器，将检查点保存到 Redis 中。

通过这些检查点器，您可以实现状态的持久化、人在回路中、时间旅行等强大功能。

### MongodbSaver[​](#mongodbsaver "MongodbSaver的直接链接")

MongodbSaver 数据库检查点器
