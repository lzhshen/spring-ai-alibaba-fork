人工介入（HITL）Hook 允许你为 Agent 工具调用添加人工监督。当模型提出需要审查的操作时——例如写入文件或执行 SQL——Hook 可以暂停执行并等待人工决策。

它通过检查每个工具调用并与可配置的策略进行比对来实现。如果需要人工干预，Hook 会发出中断（interrupt）来暂停执行。图的状态会通过 Spring AI Alibaba 的检查点机制保存，因此执行可以安全暂停并在之后恢复。

当你调用 Agent 时，它会一直运行直到完成或触发中断。当工具调用匹配你在 `approvalOn` 中配置的策略时会触发中断。在这种情况下，调用结果将返回 `InterruptionMetadata`，其中包含需要审查的操作。你可以将这些操作呈现给审查者，并在提供决策后恢复执行。

在复杂的应用场景中，你可能需要在 `StateGraph` 工作流中嵌套 `ReactAgent`，并为嵌套的 Agent 配置人工介入能力。这允许你在工作流执行过程中对 Agent 的工具调用进行人工监督。

当 `ReactAgent` 作为工作流中的一个节点时，如果 Agent 配置了 `HumanInTheLoopHook`，工作流会在 Agent 节点触发工具调用中断时暂停。工作流级别的中断处理与单独的 Agent 中断处理类似，但需要在 `CompiledGraph` 层面进行恢复。

```
import com.alibaba.cloud.ai.graph.CompileConfig;  
import com.alibaba.cloud.ai.graph.CompiledGraph;  
import com.alibaba.cloud.ai.graph.KeyStrategy;  
import com.alibaba.cloud.ai.graph.KeyStrategyFactory;  
import com.alibaba.cloud.ai.graph.NodeOutput;  
import com.alibaba.cloud.ai.graph.OverAllState;  
import com.alibaba.cloud.ai.graph.RunnableConfig;  
import com.alibaba.cloud.ai.graph.StateGraph;  
import com.alibaba.cloud.ai.graph.action.InterruptionMetadata;  
import com.alibaba.cloud.ai.graph.action.NodeAction;  
import com.alibaba.cloud.ai.graph.agent.ReactAgent;  
import com.alibaba.clo
<truncated 4547 bytes>
解释量子计算的基本原理");  
  
// 第一次调用 - 可能触发中断  
Optional<NodeOutput> nodeOutputOptional = compiledGraph.invokeAndGetOutput( // [!code highlight]  
  input,  
  RunnableConfig.builder().threadId(threadId).build()  
);  
  
// 检查是否发生中断  
if (nodeOutputOptional.isPresent()   
  && nodeOutputOptional.get() instanceof InterruptionMetadata interruptionMetadata) { // [!code highlight]  
    
  System.out.println("工作流被中断，等待人工审核。");  
  System.out.println("中断节点: " + interruptionMetadata.node());  
    
  List<InterruptionMetadata.ToolFeedback> feedbacks = interruptionMetadata.toolFeedbacks();  
    
  // 显示所有需要审批的工具调用  
  for (InterruptionMetadata.ToolFeedback feedback : feedbacks) {  
      System.out.println("工具名称: " + feedback.getName());  
      System.out.println("工具参数: " + feedback.getArguments());  
      System.out.println("工具描述: " + feedback.getDescription());  
  }  
    
  // 构建人工反馈（批准所有工具调用）  
  InterruptionMetadata.Builder feedbackBuilder = InterruptionMetadata.builder()  
      .nodeId(interruptionMetadata.node())  
      .state(interruptionMetadata.state());  
    
  feedbacks.forEach(toolFeedback -> {  
      feedbackBuilder.addToolFeedback(  
          InterruptionMetadata.ToolFeedback.builder(toolFeedback)  
              .result(InterruptionMetadata.ToolFeedback.FeedbackResult.APPROVED)  
              .build()  
      );  
  });  
    
  InterruptionMetadata approvalMetadata = feedbackBuilder.build();  
    
  // 使用批准决策恢复执行  
  RunnableConfig resumableConfig = RunnableConfig.builder()  
      .threadId(threadId) // 相同的线程ID  
      .addHumanFeedback(approvalMetadata) // [!code highlight]  
      .build();  
    
  // 恢复工作流执行（传入空Map，因为状态已保存在检查点中）  
  nodeOutputOptional = compiledGraph.invokeAndGetOutput(Map.of(), resumableConfig); // [!code highlight]  
}
```
