## Tools

许多 AI 应用程序通过自然语言与用户交互。然而，某些业务场景需要模型使用结构化输入直接与外部系统（如 API、数据库或文件系统）进行交互。

Tools 是 [agents](/docs/frameworks/agent-framework/tutorials/agents) 调用来执行操作的组件。它们通过定义良好的输入和输出让模型与外部世界交互，从而扩展模型的能力。Tools 封装了一个可调用的函数及其输入模式。我们可以把工具定义传递给兼容的 [models](/docs/frameworks/agent-framework/tutorials/models)，允许模型决定是否调用工具以及使用什么参数。在这些场景中，工具调用使模型能够生成符合指定输入模式的请求。

> **注意：服务器端工具使用**
>
> 某些聊天模型（例如 OpenAI、Anthropic 和 Gemini）具有在服务器端执行的内置工具，如 Web 搜索和代码解释器。请参阅提供商概述以了解如何使用特定聊天模型访问这些工具。

> **TIP:** 迁移与更多 Tool Calling 说明请参考：[Tool Calling 使用指南](/integration/toolcalls/tool-calls)。

*Tool calling*（也称为 *function calling*）是 AI 应用程序中的常见模式，允许 model 与一组 API 或 *tools* 交互，增强其能力。

Tools 主要用于：

* **信息检索**。此类别中的 tools 可用于从外部源检索信息，例如数据库、Web 服务、文件系统或 Web 搜索引擎。目标是增强 model 的知识，使其能够回答原本无法回答的问题。因此，它们可以在 Retrieval Augmented Generation (RAG) 场景中使用。例如，可以使用 tool 检索给定位置的当前天气、检索最新新闻文章或查询数据库中的特定记录。
* **执行操作**。此类别中的 tools 可用于在软件系统中执行操作，例如发送电子邮件、在数据库中创建新记录、提交表单或触发工作流。目标是自动化原本需要人工干预的任务，提高效率和准确性。

## Tool 核心组件[​](#tool-核心组件 "Tool 核心组件的直接链接")

### ToolCallback[​](#toolcallback "ToolCallback的直接链接")

`ToolCallback` 是 Spring AI 中用于定义工具的接口。它包含：

* **Name**: 工具的唯一标识符
* **Description**: 向模型描述工具的功能和用途
* **Input Type**: 工具接收的参数类型（通常是 Java Record 或 Class）
* **Function**: 实际执行逻辑的函数

```
public interface ToolCallback extends ToolDefinition {  
    String call(String toolInput);  
}
```

### FunctionToolCallback[​](#functiontoolcallback "FunctionToolCallback的直接链接")

这是最常用的实现，用于将通过 `java.util.function.Function` 定义的 Java 函数包装为 Tool。

```
// 1. 定义输入参数 (Record)  
public record WeatherRequest(  
   @JsonProperty(required = true, value = "location")  
   @JsonPropertyDescription("The city and state e.g. San Francisco, CA")  
   String location,  
  
   @JsonProperty(required = true, value = "unit")  
   @JsonPropertyDescription("The temperature unit: celsius or fahrenheit")  
   Unit unit) {}  
  
public enum Unit { CELSIUS, FAHRENHEIT }  
  
// 2. 定义函数逻辑  
public class WeatherService implements Function<WeatherRequest, String> {  
    @Override  
    public String apply(WeatherRequest request) {  
        // 模拟调用天气 API  
        return "25°C";  
    }  
}  
  
// 3. 构建 ToolCallback  
ToolCallback weatherTool = FunctionToolCallback  
    .builder("get_weather", new WeatherService())  
    .description("Get the weather for a location")  
    .inputType(WeatherRequest.class)  
    .build();
```

## 创建 Tools 的方式[​](#创建-tools-的方式 "创建 Tools 的方式的直接链接")

Spring AI Alibaba 提供了多种方式来创建 Tools：

### 1. 使用 `FunctionToolCallback.builder`[​](#1-使用-functiontoolcallbackbuilder "1. 使用 FunctionToolCallback.builder的直接链接")

最灵活的方式，适用于任何 Java 函数。

```
ToolCallback myTool = FunctionToolCallback  
    .builder("my_tool_name", myFunction)  
    .description("Description needed for the model")  
    .inputType(MyInput.class)  
    .build();
```

### 2. 使用 `@Tool` 注解 (Spring Bean)[​](#2-使用-tool-注解-spring-bean "2. 使用 @Tool 注解 (Spring Bean)的直接链接")

最简便的方式，适用于 Spring 托管的 Bean。Spring AI 会自动扫描带有 `@Tool` 注解的方法。

```
@Service  
public class BookService {  
  
    @Tool(description = "Search for books by author")  
    public List<Book> searchBooks(@ToolParam(description = "Author name") String author) {  
        // ...  
    }  
}
```

### 3. 使用 `Link` (Method Reference)[​](#3-使用-link-method-reference "3. 使用 Link (Method Reference)的直接链接")

可以将任何 Java 方法转换为工具，无需实现 Function 接口。

```
// 现有服务方法  
public String sendEmail(String to, String subject, String body) { ... }  
  
// 转换为 Tool  
ToolCallback emailTool = FunctionToolCallback  
    .builder("send_email", (request) -> emailService.sendEmail(request.to(), request.subject(), request.body()))  
    .description("Send an email")  
    .inputType(EmailRequest.class)  
    .build();
```

## 工具参数定义[​](#工具参数定义 "工具参数定义的直接链接")

准确的参数定义对于 LLM 正确调用工具至关重要。

### 使用 Java Records[​](#使用-java-records "使用 Java Records的直接链接")

推荐使用 Java Records，结合 `@JsonPropertyDescription` 提供参数描述。

```
public record FlightBookingRequest(  
    @JsonPropertyDescription("Origin airport code") String from,  
    @JsonPropertyDescription("Destination airport code") String to,  
    @JsonPropertyDescription("Date of flight in YYYY-MM-DD") String date  
) {}
```

### 使用 `@ToolParam`[​](#使用-toolparam "使用 @ToolParam的直接链接")

在方法参数上使用注解。

```
@Tool  
public String getStockPrice(  
    @ToolParam(description = "The stock symbol, e.g. AAPL") String symbol  
) { ... }
```

## 高级特性[​](#高级特性 "高级特性的直接链接")

### 访问上下文 (`ToolContext`)[​](#访问上下文-toolcontext "访问上下文 (ToolContext)的直接链接")

工具可以访问执行上下文，获取如用户 ID、会话 ID 等信息。

```
import org.springframework.ai.tool.ToolContext;  
  
public class PersonalAssistant implements BiFunction<String, ToolContext, String> {  
    @Override  
    public String apply(String query, ToolContext context) {  
        String userId = (String) context.getContext().get("userId");  
        // ...  
    }  
}
```

### 异常处理[​](#异常处理 "异常处理的直接链接")

工具抛出的异常会被捕获并反馈给 Agent 或 LLM。您可以自定义异常消息以引导 LLM 纠正错误。

```
if (date.isBefore(LocalDate.now())) {  
    throw new IllegalArgumentException("Cannot book flights in the past. Please provide a future date.");  
}
```

## 最佳实践[​](#最佳实践 "最佳实践的直接链接")

1. **描述清晰**：工具的 Description 和参数描述是 LLM 理解工具的关键。越详细、越准确越好。
2. **单一职责**：每个工具应该只做一件事。复杂的任务应该拆分为多个工具。
3. **容错性**：工具应该能够处理无效输入并返回有意义的错误消息。
4. **安全性**：不要让工具执行未经验证的高危操作（如删除数据），或者结合 Human-in-the-loop 进行确认。
5. **返回值**：工具的返回值应该是文本友好的，便于 LLM 理解（JSON 或清晰的字符串）。

## 示例：完整的工具定义[​](#示例完整的工具定义 "示例：完整的工具定义的直接链接")

```
import org.springframework.ai.tool.annotation.ToolParam;  
  
public record WeatherInput(  
  @ToolParam(description = "City name or coordinates") String location,  
  @ToolParam(description = "Temperature unit preference") Unit units,  
  @ToolParam(description = "Include 5-day forecast") boolean includeForecast  
) {}  
  
public enum Unit { CELSIUS, FAHRENHEIT }  
  
public class WeatherFunction implements Function<WeatherInput, String> {  
  @Override  
  public String apply(WeatherInput input) {  
      double temp = input.units() == Unit.CELSIUS ? 22 : 72;  
      String result = String.format(  
          "Current weather in %s: %.0f degrees %s",  
          input.location(),  
          temp,  
          input.units().toString().substring(0, 1).toUpperCase()  
      );  
  
      if (input.includeForecast()) {  
          result += "  
Next 5 days: Sunny";  
      }  
  
      return result;  
  }  
}  
  
ToolCallback weatherTool = FunctionToolCallback  
  .builder("get_weather", new WeatherFunction())  
  .description("Get current weather and optional forecast")  
  .inputType(WeatherInput.class)  
  .build();
```

## 访问上下文[​](#访问上下文 "访问上下文的直接链接")

**为什么这很重要**：当工具可以访问 Agent 状态、运行时上下文和长期记忆时，它们最强大。这使工具能够做出上下文感知的决策、个性化响应并在对话中维护信息。

工具可以通过 `ToolContext` 参数访问运行时信息，该参数提供：

* **State（状态）** - 通过执行流动的可变数据（消息、计数器、自定义字段）
* **Context（上下文）** - 不可变配置，如用户 ID、会话详细信息或应用程序特定配置
* **Store（存储）** - 跨对话的持久长期记忆
* **Config（配置）** - 执行的 RunnableConfig
* **Tool Call ID** - 当前工具调用的 ID

### ToolContext[​](#toolcontext "ToolContext的直接链接")

使用 `ToolContext` 在单个参数中
访问所有这些功能。

**如何使用**：

1. 实现 `BiFunction<In, ToolContext, Out>`（而不是 `Function<In, Out>`）
2. 在 `FunctionToolCallback.builder` 中使用它

```
import com.alibaba.cloud.ai.graph.agent.ReactAgent;  
import com.alibaba.cloud.ai.graph.RunnableConfig;  
import org.springframework.ai.chat.model.ToolContext;  
import java.util.function.BiFunction;  
  
// 实现 BiFunction  
public class UserAwareTool implements BiFunction<Query, ToolContext, String> {  
    @Override  
    public String apply(Query query, ToolContext context) {  
        // 1. 访问不可变上下文 (Config)  
        RunnableConfig config = (RunnableConfig) context.getContext().get("agent_config");  
        String userId = (String) config.metadata("user_id").orElse("unknown");  
          
        // 2. 访问可变状态 (State)  
        // 注意：状态访问取决于具体实现，通常通过 context 传递  
          
        return "Processing for user: " + userId;  
    }  
}  
  
// 注册工具  
ToolCallback tool = FunctionToolCallback  
    .builder("user_tool", new UserAwareTool())  
    .description("A tool that knows the user")  
    .inputType(Query.class)  
    .build();
```

## 注册工具的几种方式[​](#注册工具的几种方式 "注册工具的几种方式的直接链接")

Spring AI Alibaba 提供了多种为 Agent 注册工具的方式，适应不同的架构需求。

### `tools(ToolCallback...)`[​](#toolstoolcallback "tools(ToolCallback...)的直接链接")

最直接的方式，手动传递工具实例。

* **优点**：简单直接，完全控制。
* **缺点**：对于大量工具，构建器代码会变长。

```
ReactAgent.builder()  
    .tools(weatherTool, searchTool)  
    .build();
```

### `methodTools(Object...)`[​](#methodtoolsobject "methodTools(Object...)的直接链接")

自动扫描对象中带有 `@Tool` 注解的方法并转换为工具。

* **优点**：利用注解及反射，代码组织更整洁，类似于 Spring Controller。
* **缺点**：运行时反射开销（通常可忽略）。

```
@Service  
public class WeatherService {  
    @Tool(description = "Get weather")  
    public String getWeather(String city) { ... }  
}  
  
ReactAgent.builder()  
    .methodTools(new WeatherService())  
    .build();
```

### `toolCallbackProviders(ToolCallbackProvider...)`[​](#toolcallbackproviderstoolcallbackprovider "toolCallbackProviders(ToolCallbackProvider...)的直接链接")

使用 Provider 接口动态提供工具。

* **优点**：适合在运行时根据条件决定可用工具。

### `toolNames(String...)`[​](#toolnamesstring "toolNames(String...)的直接链接")

仅通过名称引用工具，依靠 Spring 容器或自定义解析器来查找工具 bean。

* **优点**：高度解耦，配置化。

## 总结：该选哪种方式？[​](#总结该选哪种方式 "总结：该选哪种方式？的直接链接")

| 方法 | 适用场景 | 优点 | 缺点 |
| --- | --- | --- | --- |
| `tools()` | 原型开发、少量工具 | 显式、类型安全 | 工具多时代码冗长 |
| `methodTools()` | 工具逻辑组织在类中 | 代码组织清晰、易于维护 | 需要创建工具类 |
| `toolCallbackProviders()` | 动态提供工具 | 灵活、支持运行时决策 | 需要实现接口 |
| `toolNames()` + `resolver()` | 工具定义和使用分离 | 解耦、支持配置化 | 必须配合 resolver |
| `resolver()` | 自定义解析逻辑 | 高度灵活 | 需要实现解析器 |
| 组合使用 | 复杂场景 | 最大灵活性 | 可能增加复杂度 |

### 基础使用示例[​](#基础使用示例 "基础使用示例的直接链接")

在 ReactAgent 中使用工具非常简单：

```
import com.alibaba.cloud.ai.graph.agent.ReactAgent;  
import org.springframework.ai.tool.ToolCallback;  
import org.springframework.ai.tool.function.FunctionToolCallback;  
  
// 创建工具  
ToolCallback weatherTool = FunctionToolCallback  
  .builder("get_weather", new WeatherFunction())  
  .description("Get weather for a given city")  
  .inputType(String.class)  
  .build();  
  
ToolCallback searchTool = FunctionToolCallback  
  .builder("search", new SearchFunction())  
  .description("Search for information")  
  .inputType(String.class)  
  .build();  
  
// 创建带有工具的 Agent  
ReactAgent agent = ReactAgent.builder()  
  .name("my_agent")  
  .model(chatModel)  
  .tools(weatherTool, searchTool)  
  .systemPrompt("You are a helpful assistant with access to weather and search tools.")  
  .saver(new MemorySaver())  
  .build();  
  
// 使用 Agent  
AssistantMessage response = agent.call("What's the weather like in San Francisco?");  
System.out.println(response.getText());
```

## 相关资源[​](#相关资源 "相关资源的直接链接")

* [Agents 文档](/docs/frameworks/agent-framework/tutorials/agents) - 了解如何在 Agent 中使用工具
* [Messages 文档](/docs/frameworks/agent-framework/tutorials/messages) - 了解工具消息类型
* [Models 文档](/docs/frameworks/agent-framework/tutorials/models) - 了解模型如何调用工具
