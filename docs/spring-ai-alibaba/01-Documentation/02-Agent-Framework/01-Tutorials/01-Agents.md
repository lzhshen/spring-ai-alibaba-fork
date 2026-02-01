# Agents

Agents 将大语言模型与工具结合，创建具备任务推理、工具使用决策、工具调用的自动化系统，系统具备持续推理、工具调用的循环迭代能力，直至问题解决。
Spring AI Alibaba 提供了基于 ReactAgent 的生产级 Agent 实现。

一个 LLM Agent 在循环中通过运行工具来实现目标。Agent 会一直运行直到满足停止条件 —— 即当模型输出最终答案或达到迭代限制时。

## ReactAgent 理论基础

### 什么是 ReAct

ReAct（Reasoning + Acting）是一种将推理和行动相结合的 Agent 范式。在这个范式中，Agent 会：
1. 思考（Reasoning）：分析当前情况，决定下一步该做什么
2. 行动（Acting）：执行工具调用或生成最终答案
3. 观察（Observation）：接收工具执行的结果
4. 迭代：基于观察结果继续思考和行动，直到完成任务

这个循环使 Agent 能够：
- 将复杂问题分解为多个步骤
- 动态调整策略基于中间结果
- 处理需要多次工具调用的任务
- 在不确定的环境中做出决策

### ReactAgent 的工作原理

Spring AI Alibaba 中的ReactAgent 基于 Graph 运行时构建。Graph 由节点（steps）和边（connections）组成，定义了 Agent 如何处理信息。Agent 在这个 Graph 中移动，执行如下节点：

- Model Node (模型节点)：调用 LLM 进行推理和决策
- Tool Node (工具节点)：执行工具调用
- Hook Nodes (钩子节点)：在关键位置插入自定义逻辑

## 核心组件

### Model（模型）

Model 是 Agent 的推理引擎。Spring AI Alibaba 支持多种配置方式。

#### 基础模型配置

最直接的方式是使用 ChatModel 实例：

```java
import com.alibaba.cloud.ai.dashscope.api.DashScopeApi;
import com.alibaba.cloud.ai.dashscope.chat.DashScopeChatModel;
import com.alibaba.cloud.ai.graph.agent.ReactAgent;

// 创建 DashScope API 实例
DashScopeApi dashScopeApi = DashScopeApi.builder()
        .apiKey(System.getenv("AI_DASHSCOPE_API_KEY"))
        .build();
// 创建 ChatModel
ChatModel chatModel = DashScopeChatModel.builder()
        .dashScopeApi(dashScopeApi)
        .build();
// 创建 Agent
ReactAgent agent = ReactAgent.builder()
        .name("my_agent")
        .model(chatModel)
        .build();
```

#### 高级模型配置

通过 ChatOptions 可以精细控制模型行为：

```java
import com.alibaba.cloud.ai.dashscope.chat.DashScopeChatOptions;

ChatModel chatModel = DashScopeChatModel.builder()
        .dashScopeApi(dashScopeApi)
        .defaultOptions(DashScopeChatOptions.builder()
                .withModel(DashScopeChatModel.DEFAULT_MODEL_NAME)
                .withTemperature(0.7) // 控制随机性
                .withMaxToken(2000)   // 最大输出长度
                .withTopP(0.9)        // 核采样参数
                .build())
        .build();
```

常用参数说明：
- temperature：控制输出的随机性（0.0-1.0），值越高越有创造性
- maxTokens：限制单次响应的最大 token 数
- topP：核采样，控制输出的多样性

### Tools（工具）

工具赋予 Agent 执行操作的能力，支持顺序执行、并行调用、动态选择和错误处理。

#### 定义和使用工具

```java
import org.springframework.ai.tool.ToolCallback;
import org.springframework.ai.tool.function.FunctionToolCallback;
import org.springframework.ai.tool.annotation.ToolParam;
import org.springframework.ai.chat.model.ToolContext;
import java.util.function.BiFunction;

// 定义工具（示例：仅一个搜索工具）
public class SearchTool implements BiFunction<String, ToolContext, String> {
    @Override
    public String apply(String query, ToolContext context) {
        // 实现搜索逻辑
        return "搜索结果: " + query;
    }
}

// 创建工具回调
ToolCallback searchTool = FunctionToolCallback.builder("search", new SearchTool())
        .description("搜索工具")
        .build();

// 在Agent中使用
ReactAgent agent = ReactAgent.builder()
        .name("search_agent")
        .model(chatModel)
        .tools(searchTool)
        .build();
```

#### 工具错误处理

```java
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolInterceptor;
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolCallRequest;
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolCallResponse;
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolCallHandler;

public class ToolErrorInterceptor extends ToolInterceptor {
    @Override
    public ToolCallResponse interceptToolCall(ToolCallRequest request, ToolCallHandler handler) {
        try {
            return handler.call(request);
        } catch (Exception e) {
            return ToolCallResponse.of(request.getToolCallId(), request.getToolName(), "Tool failed: " + e.getMessage());
        }
    }
    @Override
    public String getName() {
        return "ToolErrorInterceptor";
    }
}

ReactAgent agent = ReactAgent.builder()
        .name("my_agent")
        .model(chatModel)
        .interceptors(new ToolErrorInterceptor())
        .build();
```

### System Prompt（系统提示）

System Prompt 塑造 Agent 处理任务的方式。

#### 基础用法

通过 systemPrompt 参数提供字符串：

```java
ReactAgent agent = ReactAgent.builder()
        .name("my_agent")
        .model(chatModel)
        .systemPrompt("你是一个专业的技术助手。请准确、简洁地回答问题。")
        .build();
```

#### 使用 instruction

对于更详细的指令，使用 instruction 参数：

```java
String instruction = """
        你是一个经验丰富的软件架构师。
        在回答问题时，请：
        1. 首先理解用户的核心需求
        2. 分析可能的技术方案
        3. 提供清晰的建议和理由
        4. 如果需要更多信息，主动询问
        保持专业、友好的语气。
        """;

ReactAgent agent = ReactAgent.builder()
        .name("architect_agent")
        .model(chatModel)
        .instruction(instruction)
        .build();
```

#### 动态 System Prompt

使用 ModelInterceptor 实现基于上下文的动态提示：

```java
import com.alibaba.cloud.ai.graph.agent.interceptor.ModelInterceptor;
import com.alibaba.cloud.ai.graph.agent.interceptor.ModelRequest;
import com.alibaba.cloud.ai.graph.agent.interceptor.ModelResponse;
import com.alibaba.cloud.ai.graph.agent.interceptor.ModelCallHandler;
import org.springframework.ai.chat.messages.SystemMessage;

public class DynamicPromptInterceptor extends ModelInterceptor {
    @Override
    public ModelResponse interceptModel(ModelRequest request, ModelCallHandler handler) {
        // 基于上下文构建动态 system prompt
        String userRole = (String) request.getContext().getOrDefault("user_role", "default");
        String dynamicPrompt = switch (userRole) {
            case "expert" -> "你正在与技术专家对话。- 使用专业术语- 深入技术细节";
            case "beginner" -> "你正在与初学者对话。- 使用简单语言- 解释基础概念";
            default -> "你是一个专业的助手，保持友好和专业。";
        };

        SystemMessage enhancedSystemMessage;
        if (request.getSystemMessage() == null) {
            enhancedSystemMessage = new SystemMessage(dynamicPrompt);
        } else {
            enhancedSystemMessage = new SystemMessage(request.getSystemMessage().getText() + "" + dynamicPrompt);
        }

        ModelRequest modified = ModelRequest.builder(request)
                .systemMessage(enhancedSystemMessage)
                .build();
        return handler.call(modified);
    }
    @Override
    public String getName() {
        return "DynamicPromptInterceptor";
    }
}

ReactAgent agent = ReactAgent.builder()
        .name("adaptive_agent")
        .model(chatModel)
        .interceptors(new DynamicPromptInterceptor())
        .build();
```

## 调用 Agent

### 基础调用

使用 call 方法获取最终响应：

```java
import org.springframework.ai.chat.messages.AssistantMessage;

// 字符串输入
AssistantMessage response = agent.call("杭州的天气怎么样？");
System.out.println(response.getText());

// UserMessage 输入
UserMessage userMessage = new UserMessage("帮我分析这个问题");
AssistantMessage response = agent.call(userMessage);

// 多个消息
List<Message> messages = List.of(
        new UserMessage("我想了解 Java 多线程"),
        new UserMessage("特别是线程池的使用"));
AssistantMessage response = agent.call(messages);
```

### 获取完整状态

使用 invoke 方法获取完整的执行状态：

```java
import com.alibaba.cloud.ai.graph.OverAllState;
import java.util.Optional;

Optional<OverAllState> result = agent.invoke("帮我写一首诗");
if (result.isPresent()) {
    OverAllState state = result.get();
    // 访问消息历史
    Optional<Object> messages = state.value("messages");
    List<Message> messageList = (List<Message>) messages.get();
    // 访问自定义状态
    Optional<Object> customData = state.value("custom_key");
    System.out.println("完整状态：" + state);
}
```

### 使用配置

通过 RunnableConfig 传递运行时配置：

```java
import com.alibaba.cloud.ai.graph.RunnableConfig;

String threadId = "thread_123";
RunnableConfig runnableConfig = RunnableConfig.builder()
        .threadId(threadId)
        .addMetadata("key", "value")
        .build();
AssistantMessage response = agent.call("你的问题", runnableConfig);
```

## 高级特性

### 结构化输出

使用 outputType (推荐) 或 outputSchema。

```java
import org.springframework.ai.converter.BeanOutputConverter;

// 定义输出类型
public static class TextAnalysisResult {
    private String summary;
    private List<String> keywords;
    private String sentiment;
    private Double confidence;
    // Getters and Setters ...
}

ReactAgent agent = ReactAgent.builder()
        .name("analysis_agent")
        .model(chatModel)
        .outputType(TextAnalysisResult.class) // 使用 outputType
        .saver(new MemorySaver())
        .build();

AssistantMessage response = agent.call("分析这段文本：春天来了，万物复苏。");
```

### Memory（记忆）

Agent 通过状态自动维护对话历史。使用 MemorySaver 配置持久化存储。

```java
import com.alibaba.cloud.ai.graph.checkpoint.savers.MemorySaver;

// 配置内存存储
ReactAgent agent = ReactAgent.builder()
        .name("chat_agent")
        .model(chatModel)
        .saver(new MemorySaver())
        .build();

// 使用 thread_id 维护对话上下文
RunnableConfig config = RunnableConfig.builder()
        .threadId("user_123")
        .build();

agent.call("我叫张三", config);
agent.call("我叫什么名字？", config); // 输出: "你叫张三"
```

### Hooks（钩子）

Hooks 允许在 Agent 执行的关键点插入自定义逻辑。

```java
@HookPositions({HookPosition.BEFORE_MODEL})
public class MessageTrimmingHook extends MessagesModelHook {
    private static final int MAX_MESSAGES = 10;
    @Override
    public String getName() { return "message_trimming"; }
    @Override
    public AgentCommand beforeModel(List<Message> previousMessages, RunnableConfig config) {
        if (previousMessages.size() > MAX_MESSAGES) {
            // 只保留最后 MAX_MESSAGES 条消息
            List<Message> trimmedMessages = previousMessages.subList(
                    previousMessages.size() - MAX_MESSAGES,
                    previousMessages.size()
            );
            return new AgentCommand(trimmedMessages, UpdatePolicy.REPLACE);
        }
        return new AgentCommand(previousMessages);
    }
}
```

### 控制与流式输出

#### 迭代控制

```java
import com.alibaba.cloud.ai.graph.agent.hook.modelcalllimit.ModelCallLimitHook;

// 使用内置的 ModelCallLimitHook 限制模型调用次数
ReactAgent agent = ReactAgent.builder()
        .name("my_agent")
        .model(chatModel)
        .hooks(ModelCallLimitHook.builder().runLimit(5).build()) // 限制最多调用 5 次
        .saver(new MemorySaver())
        .build();
```

#### 流式输出

在 Agent 场景中，流式输出的核心是处理 `StreamingOutput` 类型。无论是模型推理、工具调用还是 Hook 节点，输出都统一为这个类型。

```java
import reactor.core.publisher.Flux;
import com.alibaba.cloud.ai.graph.NodeOutput;
import com.alibaba.cloud.ai.graph.streaming.StreamingOutput;
import com.alibaba.cloud.ai.graph.streaming.OutputType;

Flux<NodeOutput> stream = agent.stream("复杂任务");
stream.subscribe(
    output -> {
        // 检查是否为 StreamingOutput 类型
        if (output instanceof StreamingOutput streamingOutput) {
            OutputType type = streamingOutput.getOutputType();
            // 处理模型推理的流式输出
            if (type == OutputType.AGENT_MODEL_STREAMING) {
                // 流式增量内容，逐步显示
                System.out.print(streamingOutput.message().getText());
            } else if (type == OutputType.AGENT_MODEL_FINISHED) {
                // 模型推理完成，可获取完整响应
                System.out.println("\n模型输出完成");
            }
            // 处理工具调用完成
            if (type == OutputType.AGENT_TOOL_FINISHED) {
                System.out.println("工具调用完成: " + output.node());
            }
        }
    },
    error -> System.err.println("错误: " + error),
    () -> System.out.println("Agent 执行完成")
);
```
