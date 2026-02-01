# Models 模型

Spring AI Alibaba 的 ChatModel API 提供了一个统一的接口来与各种 AI 模型交互。它允许开发者在不同的模型提供商之间轻松切换，而无需更改核心代码。

## 核心接口

### ChatModel
这是主要的交互接口，扩展了 `Model<Prompt, ChatResponse>`。

```java
public interface ChatModel extends Model<Prompt, ChatResponse> {
    ChatResponse call(Prompt prompt);
    default String call(String message) { ... }
}
```

### StreamingChatModel
支持流式响应的接口，返回 `Flux<ChatResponse>`。

```java
public interface StreamingChatModel extends StreamingModel<Prompt, ChatResponse> {
    Flux<ChatResponse> stream(Prompt prompt);
    default Flux<String> stream(String message) { ... }
}
```

## DashScopeChatModel

Spring AI Alibaba 提供了 `DashScopeChatModel` 来集成阿里云通义千问系列模型。

### 依赖配置

```xml
<dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-starter-dashscope</artifactId>
</dependency>
```

### 基础使用

```java
@Autowired
private ChatModel chatModel;

// 简单调用
String response = chatModel.call("介绍一下 Spring AI Alibaba");

// 使用 Prompt 和 Options
DashScopeChatOptions options = DashScopeChatOptions.builder()
        .withModel("qwen-plus")
        .withTemperature(0.7)
        .build();

Prompt prompt = new Prompt(new UserMessage("写一个 Java HelloWorld"), options);
ChatResponse chatResponse = chatModel.call(prompt);
```

### 流式调用

```java
Flux<ChatResponse> flux = chatModel.stream(new Prompt("讲一个故事"));
flux.subscribe(response -> {
    System.out.print(response.getResult().getOutput().getText());
});
```

### 配置选项 (ChatOptions)

提供了丰富的配置选项来控制模型行为：

- `model`: 模型名称 (e.g., qwen-max, qwen-plus, qwen-turbo)
- `temperature`: 温度系数，控制随机性
- `topP`: 核采样概率
- `enableSearch`: 是否启用联网搜索 (DashScope 特性)

可以通过 `application.properties` 全局配置，也可以在 `Prompt` 中覆盖。

```yaml
spring.ai.dashscope.chat.options.model=qwen-plus
spring.ai.dashscope.chat.options.temperature=0.7
```

## 函数调用 (Function Calling)

DashScopeChatModel 完整支持 Function Calling，允许模型智能地调用注册的 Java 函数。详细信息请参考 Tools 章节。
