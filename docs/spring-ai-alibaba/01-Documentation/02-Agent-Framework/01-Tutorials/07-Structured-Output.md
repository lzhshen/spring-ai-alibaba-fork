# Structured Output 结构化输出

结构化输出允许 Agent 以特定的、可预测的格式（如 JSON）返回数据，而不是非结构化的自然语言文本。这意味着你可以将 Agent 的响应直接映射到 Java POJO。

## 核心策略

Spring AI Alibaba 支持两种主要方式来定义输出结构：

### 1. 使用 `outputType` (推荐)

直接提供 Java 类，框架会自动将其转换为 JSON Schema。这是类型安全且最简单的使用方式。

```java
public static class ContactInfo {
    private String name;
    private String email;
    private String phone;
    // Getters and Setters
}

ReactAgent agent = ReactAgent.builder()
    .model(chatModel)
    .outputType(ContactInfo.class) // 自动生成 Schema
    .build();

AssistantMessage response = agent.call("从这段文本提取联系人：张三 (zhangsan@example.com)");
// 响应将是符合 ContactInfo 结构的 JSON 字符串
```

### 2. 使用 `outputSchema`

手动提供 JSON Schema 字符串或使用 `BeanOutputConverter` 生成。

```java
BeanOutputConverter<ContactInfo> converter = new BeanOutputConverter<>(ContactInfo.class);
String schema = converter.getFormat();

ReactAgent agent = ReactAgent.builder()
    .model(chatModel)
    .outputSchema(schema)
    .build();
```

## 工作原理

Spring AI Alibaba 会根据模型的能力自动选择最佳策略：

- **原生支持**: 对于支持 Structured Output 的模型（如 OpenAI, DashScope），使用模型原生的 `response_format` 参数。
- **工具模拟**: 对于不支持原生结构化输出的模型，自动创建一个 "Format Tool"，强制模型调用该工具来输出结构化数据。
- **Prompt 增强**: 框架会自动修改 System Prompt，包含输出格式的指令。

## 错误处理

由于 LLM 的输出来源于概率生成，可能会出现格式错误。建议使用以下模式处理：

### Try-Catch 与重试

```java
try {
    AssistantMessage result = agent.call(input);
    MyData data = objectMapper.readValue(result.getText(), MyData.class);
} catch (Exception e) {
    // 可以在此处实现重试逻辑，或者要求 Agent "修复格式"
}
```

### 自定义验证

在 POJO 中添加验证逻辑：

```java
public class ContactInfo {
    public void validate() {
        if (email != null && !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email format");
        }
    }
}
```
