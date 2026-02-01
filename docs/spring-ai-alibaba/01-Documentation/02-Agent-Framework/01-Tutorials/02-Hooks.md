# Hooks 和 Interceptors

Hooks 和 Interceptors 提供了一种更精细控制 Agent 内部行为的方式。
核心 Agent 循环涉及调用模型、让其选择要执行的工具，直到不需要调用工具时完成。
Hooks 和 Interceptors 在这些步骤的前后暴露了钩子点，允许你：
- 监控: 通过日志、分析和调试跟踪 Agent 行为
- 修改: 转换提示、工具选择和输出格式
- 控制: 添加重试、回退和提前终止逻辑
- 强制执行: 应用速率限制、护栏和 PII 检测

通过将它们传递给 `ReactAgent.builder()` 来添加 Hooks 和 Interceptors：

```java
import com.alibaba.cloud.ai.graph.agent.ReactAgent;
import com.alibaba.cloud.ai.graph.agent.hook.*;
import com.alibaba.cloud.ai.graph.agent.interceptor.*;

ReactAgent agent = ReactAgent.builder()
        .name("my_agent")
        .model(chatModel)
        .tools(tools)
        .hooks(loggingHook, messageTrimmingHook)
        .interceptors(guardrailInterceptor, retryInterceptor)
        .build();
```

## 内置实现

Spring AI Alibaba 为常见用例提供了预构建的 Hooks 和 Interceptors 实现：

### 消息压缩（Summarization）

当接近 token 限制时自动压缩对话历史。

```java
import com.alibaba.cloud.ai.graph.agent.hook.summarization.SummarizationHook;

// 创建消息压缩 Hook
SummarizationHook summarizationHook = SummarizationHook.builder()
        .model(chatModel)
        .maxTokensBeforeSummary(4000)
        .messagesToKeep(20)
        .build();

// 使用
ReactAgent agent = ReactAgent.builder()
        .name("my_agent")
        .model(chatModel)
        .hooks(summarizationHook)
        .build();
```

### Human-in-the-Loop（人机协同）

暂停 Agent 执行以获得人工批准、编辑或拒绝工具调用。

```java
import com.alibaba.cloud.ai.graph.agent.hook.hip.HumanInTheLoopHook;
import com.alibaba.cloud.ai.graph.agent.hook.hip.ToolConfig;

// 创建 Human-in-the-Loop Hook
HumanInTheLoopHook humanReviewHook = HumanInTheLoopHook.builder()
        .approvalOn("sendEmailTool", ToolConfig.builder().description("Please confirm sending the email.").build())
        .approvalOn("deleteDataTool")
        .build();

ReactAgent agent = ReactAgent.builder()
        .name("supervised_agent")
        .model(chatModel)
        .tools(sendEmailTool, deleteDataTool)
        .hooks(humanReviewHook)
        .saver(new RedisSaver())
        .build();
```

### 模型调用限制（Model Call Limit）

限制模型调用次数以防止无限循环或过度成本。

```java
ReactAgent agent = ReactAgent.builder()
        .name("my_agent")
        .model(chatModel)
        .hooks(ModelCallLimitHook.builder().runLimit(5).build()) // 限制模型调用次数为5次
        .saver(new MemorySaver())
        .build();
```

### PII 检测（Personally Identifiable Information）

检测和处理对话中的个人身份信息。

```java
import com.alibaba.cloud.ai.graph.agent.hook.pii.PIIDetectionHook;
import com.alibaba.cloud.ai.graph.agent.hook.pii.PIIType;
import com.alibaba.cloud.ai.graph.agent.hook.pii.RedactionStrategy;

PIIDetectionHook pii = PIIDetectionHook.builder()
        .piiType(PIIType.EMAIL)
        .strategy(RedactionStrategy.REDACT)
        .applyToInput(true)
        .build();

// 使用
ReactAgent agent = ReactAgent.builder()
        .name("secure_agent")
        .model(chatModel)
        .hooks(pii)
        .build();
```

### 工具重试（Tool Retry）

自动重试失败的工具调用，具有可配置的指数退避。

```java
import com.alibaba.cloud.ai.graph.agent.interceptor.toolretry.ToolRetryInterceptor;

// 使用
ReactAgent agent = ReactAgent.builder()
        .name("resilient_agent")
        .model(chatModel)
        .tools(searchTool, databaseTool)
        .interceptors(ToolRetryInterceptor.builder()
                .maxRetries(2)
                .onFailure(ToolRetryInterceptor.OnFailureBehavior.RETURN_MESSAGE)
                .build())
        .build();
```

### Planning（规划）

在执行工具之前强制执行一个规划步骤，以概述 Agent 将要采取的步骤。

```java
import com.alibaba.cloud.ai.graph.agent.interceptor.todolist.TodoListInterceptor;

// 使用
ReactAgent agent = ReactAgent.builder()
        .name("planning_agent")
        .model(chatModel)
        .tools(myTool)
        .interceptors(TodoListInterceptor.builder().build())
        .build();
```

### LLM Tool Selector（LLM 工具选择器）

使用一个 LLM 来决定在多个可用工具之间选择哪个工具。

```java
import com.alibaba.cloud.ai.graph.agent.interceptor.toolselection.ToolSelectionInterceptor;

// 使用
ReactAgent agent = ReactAgent.builder()
        .name("smart_selector_agent")
        .model(chatModel)
        .tools(tool1, tool2)
        .interceptors(ToolSelectionInterceptor.builder().build())
        .build();
```

### LLM Tool Emulator（LLM 工具模拟器）

在没有实际执行工具的情况下，使用 LLM 模拟工具的输出。

```java
import com.alibaba.cloud.ai.graph.agent.interceptor.toolemulator.ToolEmulatorInterceptor;

// 使用
ReactAgent agent = ReactAgent.builder()
        .name("emulator_agent")
        .model(chatModel)
        .tools(simulatedTool)
        .interceptors(ToolEmulatorInterceptor.builder().model(chatModel).build())
        .build();
```

### Context Editing（上下文编辑）

在将上下文发送给 LLM 之前对其进行修改，以注入、删除或修改信息。

```java
import com.alibaba.cloud.ai.graph.agent.interceptor.contextediting.ContextEditingInterceptor;

// 使用
ReactAgent agent = ReactAgent.builder()
        .name("context_aware_agent")
        .model(chatModel)
        .interceptors(ContextEditingInterceptor.builder().trigger(120000).clearAtLeast(60000).build())
        .build();
```

## 自定义 Hooks 和 Interceptors

### MessagesModelHook

MessagesModelHook 是一个专门用于操作消息列表的 Hook，使用更简单，更推荐。它直接接收和返回消息列表。

```java
import com.alibaba.cloud.ai.graph.agent.hook.messages.MessagesModelHook;
import com.alibaba.cloud.ai.graph.agent.hook.messages.AgentCommand;
import com.alibaba.cloud.ai.graph.agent.hook.messages.UpdatePolicy;
import com.alibaba.cloud.ai.graph.agent.hook.HookPosition;
import com.alibaba.cloud.ai.graph.agent.hook.HookPositions;
import com.alibaba.cloud.ai.graph.RunnableConfig;
import org.springframework.ai.chat.messages.Message;

@HookPositions({HookPosition.BEFORE_MODEL})
public class MessageTrimmingHook extends MessagesModelHook {
    private static final int MAX_MESSAGES = 10;
    @Override
    public String getName() {
        return "message_trimming";
    }
    @Override
    public AgentCommand beforeModel(List<Message> previousMessages, RunnableConfig config) {
        // 如果消息数量超过限制，只保留最后 MAX_MESSAGES 条消息
        if (previousMessages.size() > MAX_MESSAGES) {
            List<Message> trimmedMessages = previousMessages.subList(
                    previousMessages.size() - MAX_MESSAGES,
                    previousMessages.size()
            );
            // 使用 REPLACE 策略替换所有消息
            return new AgentCommand(trimmedMessages, UpdatePolicy.REPLACE);
        }
        // 如果消息数量未超过限制，返回原始消息（不进行修改）
        return new AgentCommand(previousMessages);
    }
}
```

### AgentHook

在 Agent 整体执行的开始和结束时执行：

```java
import com.alibaba.cloud.ai.graph.agent.hook.AgentHook;

@HookPositions({HookPosition.BEFORE_AGENT, HookPosition.AFTER_AGENT})
public class CustomAgentHook extends AgentHook {
    @Override
    public String getName() {
        return "custom_agent_hook";
    }
    @Override
    public CompletableFuture<Map<String, Object>> beforeAgent(OverAllState state, RunnableConfig config) {
        System.out.println("Agent 开始执行");
        return CompletableFuture.completedFuture(Map.of("start_time", System.currentTimeMillis()));
    }
    @Override
    public CompletableFuture<Map<String, Object>> afterAgent(OverAllState state, RunnableConfig config) {
        System.out.println("Agent 执行完成");
        return CompletableFuture.completedFuture(Map.of());
    }
}
```

### ModelInterceptor

拦截和修改模型请求和响应：

```java
import com.alibaba.cloud.ai.graph.agent.interceptor.ModelInterceptor;
import com.alibaba.cloud.ai.graph.agent.interceptor.ModelRequest;
import com.alibaba.cloud.ai.graph.agent.interceptor.ModelResponse;
import com.alibaba.cloud.ai.graph.agent.interceptor.ModelCallHandler;

public class LoggingInterceptor extends ModelInterceptor {
    @Override
    public ModelResponse interceptModel(ModelRequest request, ModelCallHandler handler) {
        // 请求前记录
        System.out.println("发送请求到模型: " + request.getMessages().size() + " 条消息");
        // 执行实际调用
        ModelResponse response = handler.call(request);
        // 响应后记录
        System.out.println("模型响应完成");
        return response;
    }
    @Override
    public String getName() {
        return "LoggingInterceptor";
    }
}
```

#### 动态工具管理

ModelInterceptor 支持在模型调用前动态管理工具：

```java
// 动态添加工具回调
builder.dynamicToolCallbacks(dynamicTools);
// 动态筛选工具
builder.tools(List.of("search", "calculator"));
```

### ToolInterceptor

拦截和修改工具调用：

```java
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolInterceptor;

public class ToolMonitoringInterceptor extends ToolInterceptor {
    @Override
    public ToolCallResponse interceptToolCall(ToolCallRequest request, ToolCallHandler handler) {
        String toolName = request.getToolName();
        System.out.println("执行工具: " + toolName);
        try {
            ToolCallResponse response = handler.call(request);
            System.out.println("工具执行成功");
            return response;
        } catch (Exception e) {
            return ToolCallResponse.of(request.getToolCallId(), request.getToolName(), "工具执行失败: " + e.getMessage());
        }
    }
    @Override
    public String getName() {
        return "ToolMonitoringInterceptor";
    }
}
```

### 使用 RunnableConfig 跨调用共享数据

RunnableConfig 提供了一个 `context()` 方法，允许你在同一个执行流程中的不同组件间共享数据。

```java
@HookPositions({HookPosition.BEFORE_MODEL, HookPosition.AFTER_MODEL})
public class ModelCallCounterHook extends ModelHook {
    private static final String CALL_COUNT_KEY = "__model_call_count__";
    
    @Override
    public CompletableFuture<Map<String, Object>> beforeModel(OverAllState state, RunnableConfig config) {
        int currentCount = config.context().containsKey(CALL_COUNT_KEY) ? (int) config.context().get(CALL_COUNT_KEY) : 0;
        System.out.println("模型调用 #" + (currentCount + 1));
        return CompletableFuture.completedFuture(Map.of());
    }
    
    @Override
    public CompletableFuture<Map<String, Object>> afterModel(OverAllState state, RunnableConfig config) {
        int currentCount = config.context().containsKey(CALL_COUNT_KEY) ? (int) config.context().get(CALL_COUNT_KEY) : 0;
        config.context().put(CALL_COUNT_KEY, currentCount + 1);
        return CompletableFuture.completedFuture(Map.of());
    }
}
```

## 执行顺序

1. **Before Agent Hooks** (顺序)
2. **Agent 循环开始**
3. **Before Model Hooks** (顺序)
4. **Model Interceptors** (嵌套: 1 -> 2 -> Model)
5. **After Model Hooks** (逆序)
6. **Tool Interceptors** (嵌套)
7. **Agent 循环结束**
8. **After Agent Hooks** (逆序)
