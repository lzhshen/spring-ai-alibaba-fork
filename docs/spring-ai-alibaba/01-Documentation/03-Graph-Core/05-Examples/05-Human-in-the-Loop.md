# 人类反馈 (Human-in-the-Loop)

在构建 Agent 工作流时，人类介入是一个核心场景。Spring AI Alibaba 提供两种模式。

## 模式一：interruptBefore (被动中断)

在编译图时指定哪些节点需要人工预审。

```java
var compileConfig = CompileConfig.builder()
    .saverConfig(saverConfig)
    .interruptBefore("human_review_node") // 运行到此节点前自动挂起
    .build();

CompiledGraph app = workflow.compile(compileConfig);
```

## 模式二：InterruptionMetadata (主动中断)

节点内部根据逻辑主动触发中断。

```java
public class MyNode implements InterruptableNodeAction {
    @Override
    public Map<String, Object> apply(OverAllState state) throws Exception {
        if (needApproval(state)) {
            // 返回中断元数据
            return Map.of("_interruption", InterruptionMetadata.of("需要人工审批"));
        }
        return Map.of("status", "done");
    }
}
```
