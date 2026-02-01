# 状态回溯 (Time Travel)

利用持久化能力，你可以将工作流恢复到历史的任何一个检查点。

```java
// 获取历史快照列表
List<StateSnapshot> history = graph.getStateHistory(config);

// 恢复到第 3 步的状态
RunnableConfig pastConfig = history.get(2).config();
graph.stream(null, pastConfig); // 从该点重新开始执行
```
