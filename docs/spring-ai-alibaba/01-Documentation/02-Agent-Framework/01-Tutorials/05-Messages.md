# Messages 消息

Messages 是 Spring AI Alibaba 中模型交互的基本单元。它们封装了角色、内容和元数据，提供了跨不同 LLM 提供商的一致抽象。

## 消息类型

Spring AI 使用以下主要消息类型：

- **SystemMessage**: 设定 AI 的角色、行为准则和上下文。
- **UserMessage**: 代表用户的输入，可以是文本、图像、音频或视频。
- **AssistantMessage**: 代表 AI 的响应，包含文本内容或工具调用请求。
- **ToolResponseMessage**: 包含工具执行的结果，用于回传给模型。

## 基础使用

```java
import org.springframework.ai.chat.messages.*;
import org.springframework.ai.chat.prompt.Prompt;

List<Message> messages = List.of(
    new SystemMessage("你是一个专业的 Java 助手。"),
    new UserMessage("如何创建一个 Spring Boot 应用？")
);

Prompt prompt = new Prompt(messages);
ChatResponse response = chatModel.call(prompt);
```

### 使用 Builder

所有消息类型都支持 Builder 模式：

```java
UserMessage userMsg = UserMessage.builder()
    .text("分析这张图片")
    .media(new Media(MimeTypeUtils.IMAGE_JPEG, new ClassPathResource("image.jpg")))
    .metadata(Map.of("userId", "123"))
    .build();
```

## 多模态支持

UserMessage 支持多种媒体类型作为输入：

### 图像

```java
UserMessage message = UserMessage.builder()
    .text("这张图片里有什么？")
    .media(Media.builder()
        .mimeType(MimeTypeUtils.IMAGE_JPEG)
        .data(new URL("https://example.com/image.jpg"))
        .build())
    .build();
```

### 音频与视频

类似地，可以使用 `MimeTypeUtils` 指定音频 (`audio/wav`) 或视频 (`video/mp4`) 类型。

## 工具调用与响应

当模型决定调用工具时，它会返回包含 `toolCalls` 的 `AssistantMessage`。执行工具后，你需要创建一个 `ToolResponseMessage` 并将其添加回对话历史中。

```java
// 1. 模型返回工具调用
AssistantMessage aiMsg = response.getResult().getOutput();
List<ToolCall> toolCalls = aiMsg.getToolCalls();

// 2. 执行工具并创建响应消息
List<ToolResponse> responses = new ArrayList<>();
for (ToolCall call : toolCalls) {
    String result = myTool.execute(call.arguments());
    responses.add(new ToolResponse(call.id(), call.name(), result));
}
ToolResponseMessage toolMsg = new ToolResponseMessage(responses);

// 3. 将 AI 消息和工具响应消息都添加到历史中，再次调用模型
List<Message> history = new ArrayList<>(originalMessages);
history.add(aiMsg);
history.add(toolMsg);

ChatResponse finalResponse = chatModel.call(new Prompt(history));
```

## 在 ReactAgent 中使用

虽然 ReactAgent 自动管理消息循环，但你可以直接传递消息来启动对话：

```java
AssistantMessage response = agent.call(new UserMessage("你好"));
```

或者传递消息历史列表：

```java
List<Message> history = List.of(
    new UserMessage("设置上下文..."),
    new UserMessage("提出请求...")
);
AssistantMessage response = agent.call(history);
```
