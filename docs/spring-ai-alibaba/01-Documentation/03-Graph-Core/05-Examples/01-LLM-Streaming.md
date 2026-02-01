# LLM 流式输出

在节点内部集成 LLM 的流式能力，并实时推送到 Graph 的输出流中。

```java
graph.stream(Map.of("prompt", "讲个故事"), config)
    .doOnNext(output -> {
        if (output instanceof StreamingOutput<?> streaming) {
            // 获取实时 Token
            System.out.print(streaming.message().getText());
        }
    }).subscribe();
```
