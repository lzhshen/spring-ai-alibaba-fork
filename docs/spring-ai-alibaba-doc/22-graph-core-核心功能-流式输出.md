## 流式输出

Spring AI Alibaba Graph 内置了对流式处理的原生支持，框架统一是使用 Flux 来在框架中定义和传递流，与 Spring 生态的流式处理保持一致。以下是从 Graph 运行中流式返回输出的不同方式。

## 调用 Graph 的流式输出[​](#调用-graph-的流式输出 "调用 Graph 的流式输出的直接链接")

`.stream()` 是一个用于从图运行中流式返回输出的方法。它返回一个 Flux，请记住由于 Flux 流式的特性，流返回后并不会立即出触发图引擎的执行，你需要执行类似 Flux.subscribe() 的操作才能真正启动流引擎。

目前 Flux 返回的是 `NodeOutput` 类的实例，该类基本上报告执行的**节点名称**和结果**状态**。

### 流的组合（嵌入和组合）[​](#流的组合嵌入和组合 "流的组合（嵌入和组合）的直接链接")

Flux 支持多个流的合并、转换、组合等操作，具备非常强大的能力，这在处理图中多个流式节点时会非常有用。具体使用方式可搜索 Spring Reactor 学习。

## 理解[​](#理解 "理解的直接链接")

## 在节点操作中整合流式输出[​](#在节点操作中整合流式输出 "在节点操作中整合流式输出的直接链接")

在 Spring AI Alibaba Graph 中，您可以在节点操作中直接返回 `Flux` 对象，框架会自动处理流式输出。

### 流式节点实现[​](#流式节点实现 "流式节点实现的直接链接")

```
import com.alibaba.cloud.ai.graph.OverAllState;  
import com.alibaba.cloud.ai.graph.action.NodeAction;  
import org.springframework.ai.chat.client.ChatClient;  
import org.springframework.ai.chat.model.ChatResponse;  
  
import java.util.Map;  
  
import reactor.core.publisher.Flux;  
  
/**  
* 流式节点实现 - 使用 GraphFluxGenerator 处理流式响应  
*/  
public static class StreamingNode implements NodeAction {  
  
  private final ChatClient chatClient;  

<truncated 14226 bytes>
的输出，NodeOutput 中包含整个图的当前全局 OverallState 状态、当前节点的 Message 输出等，不同的节点可能返回不同子类型：

**类型层次：**

```
// 基类：所有节点输出的基础类型  
NodeOutput {  
  - node: String              // 节点 ID  
  - state: OverallState      // 全局状态  
  - message: Object          // 节点消息  
}  
  
// 子类型 1：LLM 流式输出（框架内置）  
StreamingOutput extends NodeOutput {  
  - chunk: String            // 流式 Token 内容  
}  
  
// 子类型 2：普通节点输出（框架默认）  
NodeOutput (普通实例)  
  
// 子类型 3：用户自定义（可扩展）  
CustomOutput extends NodeOutput {  
  - customField: Object      // 用户自定义字段  
}
```

**使用场景：**

* **StreamingOutput**：框架自动为 LLM 流式节点创建，标识流式 Token 输出块
* **NodeOutput**：普通节点的标准输出类型
* **自定义类型**：用户可以基于 `NodeOutput` 扩展任意类型，在自定义节点中返回

## 并行节点的流式输出[​](#并行节点的流式输出 "并行节点的流式输出的直接链接")

如果你有多个并行节点（普通节点或者嵌套子图），可以参考 [并行节点的流式处理](/docs/frameworks/graph-core/examples/parallel-streaming) 来了解详情。

## 最佳实践[​](#最佳实践 "最佳实践的直接链接")

1. **使用适当的订阅方式**：根据需求选择 `subscribe()`、`blockLast()` 或其他 Reactor 操作符
2. **错误处理**：始终使用 `doOnError()` 处理流式输出中的错误
3. **资源清理**：确保在流完成或取消时正确清理资源
4. **性能考虑**：对于大量数据，使用背压（backpressure）机制控制流的速度

## 相关文档[​](#相关文档 "相关文档的直接链接")

* [快速入门](/docs/frameworks/graph-core/quick-start) - Graph 基础使用
* [Spring Reactor 文档](https://projectreactor.io/docs/core/release/reference/) - Reactor 流式处理参考
