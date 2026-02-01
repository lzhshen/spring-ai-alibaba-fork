## Hooks 和 Interceptors

> 让开发者在每个步骤控制和自定义 Agent 执行

Hooks 和 Interceptors 提供了一种更精细控制 Agent 内部行为的方式。

核心 Agent 循环涉及调用模型、让其选择要执行的工具，直到不需要调用工具时完成。

![reactagent](/img/agent/agents/reactagent.png)

Hooks 和 Interceptors 在这些步骤的前后暴露了钩子点，允许你：

![reactagent](/img/agent/hooks/reactagent-hook.png)

* **监控**: 通过日志、分析和调试跟踪 Agent 行为
* **修改**: 转换提示、工具选择和输出格式
* **控制**: 添加重试、回退和提前终止逻辑
* **强制执行**: 应用速率限制、护栏和 PII 检测

通过将它们传递给 `ReactAgent.builder()` 来添加 Hooks 和 Interceptors：

```
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

## Hooks 和 Interceptors 能做什么？[​](#hooks-和-interceptors-能做什么 "Hooks 和 Interceptors 能做什么？的直接链接")

* 监控。使用日志、分析和调试跟踪 Agent 行为。
* 修改。转换提示、工具选择和输出格式。
* 控制。添加重试、回退和提前终止逻辑。
* 强制执行。应用速率限制、护栏和 PII 检测。

## 内置实现[​](#内置实现 "内置实现的直接链接")

Spring AI Alibaba 为常见用例提供了预构建的 Hooks 和 Interceptors 实现：

### 消息压缩（Summarization）[​](#消息压缩summarization "消息压缩（Summarization）的直接链接")

当接近 token 限制时自动压缩对话历史。

**适用场景**：

* 超出上下文窗口的长期对话
* 保持 Agent 聚焦于最近的上下文
* 降低 Token 成本

```
import com.alibaba.cloud.ai.graph.agent.hook.messages.MessageSummarizerHook;  
  
MessageSummarizerHook hook = MessageSummarizerHook.builder()  
  .maxRetainedMessages(10) // 保留最后 10 条消息  
  .build();  
  
// 使用  
ReactAgent agent = ReactAgent.builder()  
  .name("long_running_agent")  
  .model(chatModel)  
  .hooks(hook)  
  .build();
```

### PII 检测与脱敏[​](#pii-检测与脱敏 "PII 检测与脱敏的直接链接")

检测并屏蔽敏感信息（如电子邮件、电话号码）。

**适用场景**：

* 处理用户输入的隐私合规
* 防止敏感数据泄露给 LLM 提供商
* 记录脱敏后的日志

```
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

### 工具重试（Tool Retry）[​](#工具重试tool-retry "工具重试（Tool Retry）的直接链接")

自动重试失败的工具调用，具有可配置的指数退避。

**适用场景**：

* 处理外部 API 调用中的瞬态故障
* 提高依赖网络的工具的可靠性
* 构建优雅处理临时错误的弹性 Agent

```
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

### Planning（规划）[​](#planning规划 "Planning（规划）的直接链接")

在执行工具之前强制执行一个规划步骤，以概述 Agent 将要采取的步骤。

**适用场景**：

* 需要执行复杂、多步骤任务的 Agent
* 通过在执行前显示 Agent 的计划来提高透明度
* 通过检查建议的计划来调试错误

```
import com.alibaba.cloud.ai.graph.agent.interceptor.todolist.TodoListInterceptor;  
  
// 使用  
ReactAgent agent = ReactAgent.builder()  
  .name("planning_agent")  
  .model(chatModel)  
  .tools(myTool)  
  .interceptors(TodoListInterceptor.builder().build())  
  .build();
```

### LLM Tool Selector（LLM 工具选择器）[​](#llm-tool-selectorllm-工具选择器 "LLM Tool Selector（LLM 工具选择器）的直接链接")

使用一个 LLM 来决定在多个可用工具中选择哪一个。这对于当有大量工具可用时，帮助模型准确选择最有用的工具非常有用。

### 自定义 Hooks[​](#自定义-hooks "自定义 Hooks的直接链接")

您可以创建自己的 Hook 来实现特定的业务逻辑。

```
import com.alibaba.cloud.ai.graph.agent.hook.messages.MessagesModelHook;  
import com.alibaba.cloud.ai.graph.agent.hook.HookPosition;  
import com.alibaba.cloud.ai.graph.agent.hook.HookPositions;  
import com.alibaba.cloud.ai.graph.agent.hook.messages.AgentCommand;  
import com.alibaba.cloud.ai.graph.RunnableConfig;  
import com.alibaba.cloud.ai.graph.OverAllState;  
import org.springframework.ai.chat.messages.Message;  
import java.util.concurrent.CompletableFuture;  
import java.util.List;  
import java.util.Map;  
  
@HookPositions({HookPosition.BEFORE_MODEL, HookPosition.AFTER_MODEL})  
public class ModelCallCounterHook extends MessagesModelHook {  
  
  private static final String CALL_COUNT_KEY = "__model_call_count__";  
  
  @Override  
  public String getName() {  
      return "model_call_counter";  
  }  
  
  @Override  
  public AgentCommand beforeModel(List<Message> previousMessages, RunnableConfig config) {  
      // 检查调用次数  
      int callCount = config.context().containsKey(CALL_COUNT_KEY)  
              ? (int) config.context().get(CALL_COUNT_KEY) : 0;  
      
      // 限制逻辑示例，实际应使用 ModelCallLimitHook  
      int maxCalls = 5;  
      if (callCount >= maxCalls) {  
          // 返回终止指令，不再调用模型  
          return new AgentCommand(previousMessages, AgentCommand.CommandType.ABORT);  
          // 或者修改消息通知 Agent 结束，这里简单返回。  
          // 更好的做法是修改消息历史加入系统提示让其结束，或者直接抛出异常  
  
          // 为了演示如何终止循环，我们可以注入一条助手的消息说"任务因限制停止"  
          // 这里的 AgentCommand 支持修改消息列表  
  
          List<Message> messages = new ArrayList<>(  
              (List<Message>) state.value("messages").orElse(new ArrayList<>())  
          );  
          messages.add(new AssistantMessage(  
              "已达到模型调用次数限制 (" + callCount + "/" + maxCalls + ")，Agent 执行终止。"  
          ));  
  
          // 返回更新并跳转到结束  
          return CompletableFuture.completedFuture(Map.of("messages", messages));  
      }  
  
      return CompletableFuture.completedFuture(Map.of());  
  }  
  
  @Override  
  public CompletableFuture<Map<String, Object>> afterModel(OverAllState state, RunnableConfig config) {  
      // 递增计数器  
      int callCount = config.context().containsKey(CALL_COUNT_KEY)  
              ? (int) config.context().get(CALL_COUNT_KEY) : 0;  
      config.context().put(CALL_COUNT_KEY, callCount + 1);  
  
      return CompletableFuture.completedFuture(Map.of());  
  }  
  
  @Override  
  public List<JumpTo> canJumpTo() {  
      return List.of(JumpTo.end);  
  }  
}
```

**使用示例**：

```
ReactAgent agent = ReactAgent.builder()  
  .name("limited_agent")  
  .model(chatModel)  
  .tools(tools)  
  .hooks(new ModelCallCounterHook())  // 监控调用统计  
  .hooks(new ModelCallLimiterHook(5)) // 限制最多调用 5 次  
  .build();
```

**关键要点**：

* **context() 是共享的**: 同一个执行流程中的所有 Hook 共享同一个 context
* **数据持久性**: context 中的数据在整个 Agent 执行期间保持有效
* **类型安全**: 需要自己管理 context 中数据的类型转换
* **命名约定**: 建议使用双下划线前缀命名 context key（如 `__model_call_count__`）以避免与用户数据冲突

## 执行顺序[​](#执行顺序 "执行顺序的直接链接")

使用多个 Hooks 和 Interceptors 时，理解执行顺序很重要：

```
ReactAgent agent = ReactAgent.builder()  
  .name("my_agent")  
  .hooks(hook1, hook2) // 顺序：hook1 -> hook2  
  .interceptors(interceptor1, interceptor2) // 顺序：interceptor1 -> interceptor2  
  .build();
```

通过合理组合 Hooks 和 Interceptors，您可以构建出强大、可观察且安全的 Agent 系统。
