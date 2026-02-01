# 流式输出 (Streaming)

Spring AI Alibaba Graph 原生支持流式响应，能够实时返回每个节点的执行状态以及 LLM 生成的各个 Token。

## 核心 API

### .stream()
使用 `.stream()` 替代 `.invoke()`。它返回一个 `Flux<NodeOutput>`。
> [!NOTE]
> 这是一个惰性流，必须进行订阅 (如 `.subscribe()` 或 `.blockLast()`) 才会真正启动执行。

```java
Flux<NodeOutput> stream = graph.stream(input, config);

stream.doOnNext(output -> {
    System.out.println("节点: " + output.node());
    System.out.println("状态更新: " + output.state().data());
}).subscribe();
```

## 流的层次结构

1. **节点级流 (Node Output)**: 每当一个节点完成工作，就会发布一个 `NodeOutput`。
2. **Token 级流 (LLM Stream)**: 当节点内部调用 LLM 的流式接口时，框架可以将生成的 Token 包装为 `StreamingOutput` 实时传回给客户端。

## 流式节点实现

在自定义节点中，你可以直接返回 `Flux`。

```java
public class MyStreamingNode implements NodeAction {
    @Override
    public Map<String, Object> apply(OverAllState state) {
        // 返回 Flux 给框架处理
        Flux<String> tokenFlux = chatClient.prompt().stream().content();
        return Map.of("content_flux", tokenFlux);
    }
}
```

## 处理流式输出 (客户端)

```java
graph.stream(input, config)
    .doOnNext(output -> {
        if (output instanceof StreamingOutput<?> streaming) {
            // 这是来自 LLM 的实时 Token
            System.out.print(streaming.message().getText());
        } else {
            // 普通节点的状态转换输出
            System.out.println("\n--- 节点完成: " + output.node());
        }
    })
    .subscribe();
```

## 最佳实践
- **UI 响应**: 对于带有 LLM 的工作流，始终使用流式输出以提升用户体验，避免长达数十秒的“黑盒”等待。
- **背压控制**: 使用 Reactive Streams 的特性（如 `limitRate`）来管理高频状态更新。
