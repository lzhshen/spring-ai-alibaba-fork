# Tools 工具

Tools 是 Agent 调用来执行操作的组件。它们通过定义良好的输入和输出让模型与外部世界交互，从而扩展模型的能力。
Tools 封装了一个可调用的函数及其输入模式。我们可以把工具定义传递给兼容的模型，允许模型决定是否调用工具以及使用什么参数。

## 快速开始

### 1. 定义工具

你可以使用 `@Tool` 注解将方法定义为工具：

```java
import java.time.LocalDateTime;
import org.springframework.ai.tool.annotation.Tool;
import org.springframework.context.i18n.LocaleContextHolder;

class DateTimeTools {
    @Tool(description = "Get the current date and time in the user's timezone")
    String getCurrentDateTime() {
        return LocalDateTime.now().atZone(LocaleContextHolder.getTimeZone().toZoneId()).toString();
    }
}
```

### 2. 使用工具

将工具实例传递给 `ChatClient` 或 `ReactAgent`：

```java
ChatModel chatModel = ...;
String response = ChatClient.create(chatModel)
        .prompt("What day is tomorrow?")
        .tools(new DateTimeTools())
        .call()
        .content();
System.out.println(response);
```

## 创建工具

Spring AI 提供了三种主要方式来创建工具：

### 1. 方法作为 Tools (@Tool)

这是最简单的方法，适用于将现有服务方法公开为工具。

```java
public class CalculatorTools {
    @Tool(description = "Add two numbers together")
    public String add(
            @ToolParam(description = "First number") int a,
            @ToolParam(description = "Second number") int b) {
        return String.valueOf(a + b);
    }
}
```

- `@Tool`: 标记方法为工具。`description` 是必需的，用于帮助模型理解工具的用途。
- `@ToolParam`: 可选。用于描述参数用途和是否必需。

### 2. 函数作为 Tools (FunctionCallback)

通过编程方式将 `Function`, `Supplier`, `Consumer` 或 `BiFunction` 包装为工具。

```java
import org.springframework.ai.tool.function.FunctionToolCallback;

ToolCallback weatherTool = FunctionToolCallback
        .builder("currentWeather", new WeatherService())
        .description("Get the weather in location")
        .inputType(WeatherRequest.class)
        .build();
```

### 3. Bean 作为 Tools

将 Bean 定义为工具，Spring AI 会自动解析。

```java
@Configuration(proxyBeanMethods = false)
class WeatherTools {
    @Bean
    @Description("Get the weather in location")
    Function<WeatherRequest, WeatherResponse> currentWeather() {
        return new WeatherService();
    }
}
```

## 在 ReactAgent 中使用工具

ReactAgent 提供了灵活的方式来集成工具：

### 1. 直接传递 (tools)

直接传递 `ToolCallback` 实例。

```java
ReactAgent agent = ReactAgent.builder()
        .tools(weatherTool, searchTool)
        .build();
```

### 2. 方法工具 (methodTools)

传递包含 `@Tool` 注解方法的对象实例。

```java
ReactAgent agent = ReactAgent.builder()
        .methodTools(new CalculatorTools())
        .build();
```

### 3. 工具提供者 (toolCallbackProviders)

传递 `ToolCallbackProvider` 实现，用于动态提供工具。

```java
ReactAgent agent = ReactAgent.builder()
        .toolCallbackProviders(new CustomToolProvider())
        .build();
```

### 4. 工具名称与解析器 (toolNames + resolver)

指定工具名称，并提供解析器来查找工具。

```java
ReactAgent agent = ReactAgent.builder()
        .toolNames("calculator", "search")
        .resolver(myResolver)
        .build();
```

## 访问上下文 (ToolContext)

工具可以通过 `ToolContext` 访问运行时信息、Agent 状态和配置，而无需将这些信息暴露给 LLM。

```java
import org.springframework.ai.chat.model.ToolContext;

public class UserProfileTool implements BiFunction<String, ToolContext, String> {
    @Override
    public String apply(String userId, ToolContext context) {
        // 访问配置
        RunnableConfig config = (RunnableConfig) context.getContext().get("config");
        // 访问 Agent 状态
        OverAllState state = (OverAllState) context.getContext().get("state");
        
        return "User Profile for " + userId;
    }
}
```

## 高级配置

### 直接返回 (Return Direct)

如果希望工具执行后直接将结果返回给用户，而不是让 LLM 继续生成响应，可以设置 `returnDirect = true`。

```java
@Tool(description = "...", returnDirect = true)
public String specificAction() { ... }
```

或者在编程式定义中：

```java
ToolMetadata metadata = ToolMetadata.builder().returnDirect(true).build();
```

### 异常处理

当工具执行失败时，默认行为是将错误消息作为工具输出返回给模型，让模型决定如何处理（例如重试或通知用户）。可以通过 `ToolExecutionExceptionProcessor` 自定义此行为。
