# Structured Output (结构化输出)

Spring AI Alibaba 支持将 LLM 的文本输出转换为强类型的 Java 对象，便于程序处理。

## 核心组件

-   **StructuredOutputConverter<T>**: 负责转换逻辑的核心接口，继承自 `Converter<String, T>` 和 `FormatProvider`。
-   **BeanOutputConverter**: 使用 "DRAFT_2020_12" schema 将 JSON 反序列化为 POJO。
-   **MapOutputConverter**: 将响应生成为 `java.util.Map<String, Object>`。
-   **ListOutputConverter**: 将逗号分隔的文本转换为 `java.util.List`。

## 工作原理

1.  **Prompt 注入**: 在调用前，转换器将格式指令（Format Instructions）附加到提示中，定义所需的 Schema。
2.  **解析转换**: 调用后，转换器获取模型的输出文本并将其转换为结构化类型的实例。

## 代码示例

### Fluent API

```java
ActorsFilms result = ChatClient.create(chatModel).prompt()
    .user("Generate films for Tom Hanks")
    .call()
    .entity(ActorsFilms.class);
```

### Low-level API

工具执行转换操作：

```java
this.beanOutputConverter.convert(this.generation.getOutput().getText());
```

## 配置

Providers 支持通过属性配置内置 JSON 模式，例如设置 `spring.ai.openai.chat.options.responseFormat` 为 `json_object`。
