# 状态持久化 (Persistence)

持久化允许图工作流在执行期间保存快照，支持跨请求恢复、错误重试以及人工介入 (Human-in-the-loop) 场景。

## 会话 (Session)

会话由 `threadId` 标识。每次运行图时，底层状态都会自动持久化到该会话中。
- **历史回溯**: 可以访问会话的所有历史版本。
- **并发控制**: 确保同一 `threadId` 的执行是串行的。

## 检查点 (Checkpoints)

检查点是状态的快照 (`StateSnapshot`)，在每个超级步骤 (Superstep) 完成后自动保存。
快照包含：
- **values**: 当前所有状态通道的值。
- **next**: 下一个待执行节点的名称。
- **config/metadata**: 运行时配置和元数据。

## 检查点器 (Checkpointer) 实现

Spring AI Alibaba 提供了多种后端支持：

1. **MemorySaver**: 内存存储，适用于测试和无状态服务负载。
2. **PostgreSqlSaver**: 专用于 PostgreSQL，提供跨进程持久化。
3. **RedisSaver**: 基于 Redis，适用于高并发场景。
4. **MongodbSaver**: 基于 MongoDB。

### 配置示例

```java
SaverConfig saverConfig = SaverConfig.builder()
    .register(new PostgreSqlSaver(dataSource))
    .build();

CompiledGraph graph = workflow.compile(
    CompileConfig.builder().saverConfig(saverConfig).build()
);
```

## 核心操作

- **getState**: 获取指定 `threadId` 的最新快照。
- **getStateHistory**: 列出所有历史快照。
- **updateState**: 手动修改当前快照的状态或跳转到特定节点。

## 人工介入模式 (Human-in-the-loop)

持久化是人工介入的基础。通过在特定节点前设置中断点，状态会被保存并暂停执行，直到人工干预。

```java
CompileConfig config = CompileConfig.builder()
    .interruptBefore("approval_node")
    .build();
```
