## Agents

Agents 将大语言模型与工具结合，创建具备任务推理、工具使用决策、工具调用的自动化系统，系统具备持续推理、工具调用的循环迭代能力，直至问题解决。

Spring AI Alibaba 提供了基于 `ReactAgent` 的生产级 Agent 实现。

**一个 LLM Agent 在循环中通过运行工具来实现目标**。Agent 会一直运行直到满足停止条件 —— 即当模型输出最终答案或达到迭代限制时。

## ReactAgent 理论基础[​](#reactagent-理论基础 "ReactAgent 理论基础的直接链接")

### 什么是 ReAct[​](#什么是-react "什么是 ReAct的直接链接")

ReAct（Reasoning + Acting）是一种将推理和行动相结合的 Agent 范式。在这个范式中，Agent 会：

1. **思考（Reasoning）**：分析当前情况，决定下一步该做什么
2. **行动（Acting）**：执行工具调用或生成最终答案
3. **观察（Observation）**：接收工具执行的结果
4. **迭代**：基于观察结果继续思考和行动，直到完成任务

这个循环使 Agent 能够：

* 将复杂问题分解为多个步骤
* 动态调整策略基于中间结果
* 处理需要多次工具调用的任务
* 在不确定的环境中做出决策

### ReactAgent 的工作原理[​](#reactagent-的工作原理 "ReactAgent 的工作原理的直接链接")

Spring AI Alibaba 中的`ReactAgent` 基于 **Graph 运行时**构建。Graph 由节点（steps）和边（connections）组成，定义了 Agent 如何处理信息。Agent 在这个 Graph 中移动，执行如下节点：

* **Model Node (模型节点)**：调用 LLM 进行推理和决策
* **Tool Node (工具节点)**：执行工具调用
* **Hook Nodes (钩子节点)**：在关键位置插入自定义逻辑

ReactAgent 的核心执行流程：

![reactagent](/img/agent/agents/reactagent.png)

## 核心组件[​](#核心组件 "核心组件的直接链接")

### Model（模型）[​](#model "Models的直接链接")

Model 是 Agent 的大脑。它负责：

* 接收当前状态（对话历史、工具结果等）
* 决定下一步行动（调用工具或结束对话）
* 生成响应内容

```
ChatModel chatModel = ... // 初始化 ChatModel  
  
ReactAgent agent = ReactAgent.builder()  
  .name("my_agent")  
  .model(chatModel)  
  .build();
```

### Tools（工具）[​](#tools "Tools的直接链接")

工具是 Agent 与外部世界交互的方式。Agent 可以调用工具来获取信息或执行操作。

```
// 定义工具  
ToolCallback weatherTool = FunctionToolCallback  
    .builder("get_weather", new WeatherService())  
    .description("获取指定城市的天气")  
    .build();  
  
ReactAgent agent = ReactAgent.builder()  
  .name("weather_agent")  
  .model(chatModel)  
  .tools(weatherTool)  
  .build();
```

### Memory（记忆）[​](#memory "Memory的直接链接")

记忆使 Agent 能够在多个回合中保持上下文。

```
ReactAgent agent = ReactAgent.builder()  
  .name("memory_agent")  
  .model(chatModel)  
  .saver(new MemorySaver()) // 启用记忆  
  .build();
```

### Hooks & Interceptors（钩子与拦截器）[​](#hooks--interceptors "Hooks & Interceptors的直接链接")

用于扩展和定制 Agent 的行为。

```
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolErrorInterceptor;  
import org.springframework.ai.chat.messages.ToolCallRequest;  
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolCallResponse;  
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolCallHandler;  
  
public class ToolErrorInterceptor extends ToolInterceptor {  
  @Override  
  public ToolCallResponse interceptToolCall(ToolCallRequest request, ToolCallHandler handler) {  
      try {  
          return handler.call(request);  
      } catch (Exception e) {  
          return ToolCallResponse.of(request.getToolCallId(), request.getToolName(),  
              "Tool failed: " + e.getMessage());  
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

**ReAct 循环示例**：Agent 自动交替进行推理和工具调用，直到获得最终答案。

```
用户: 查询杭州天气并推荐活动  
→ [推理] 需要查天气 → [行动] get_weather("杭州") → [观察] 晴，25°C  
→ [推理] 需要推荐活动 → [行动] search("户外活动") → [观察] 西湖游玩...  
→ [推理] 信息充足 → [行动] 生成答案
```

### System Prompt（系统提示）[​](#system-prompt系统提示 "System Prompt（系统提示）的直接链接")

System Prompt 塑造 Agent 处理任务的方式。

#### 基础用法[​](#基础用法 "基础用法的直接链接")

通过 `systemPrompt` 参数提供字符串：

```
ReactAgent agent = ReactAgent.builder()  
  .name("my_agent")  
  .model(chatModel)  
  .systemPrompt("你是一个专业的技术助手。请准确、简洁地回答问题。")  
  .build();
```

#### 使用 instruction[​](#使用-instruction "使用 instruction的直接链接")

对于更详细的指令，使用 `instruction` 参数：

```
String instruction = """  
  你是一个经验丰富的软件架构师。  
  
  在回答问题时，请：  
  1. 首先理解用户的需求
  2. 提供多个解决方案
  3. 分析每个方案的优缺点
  """;  
  
ReactAgent agent = ReactAgent.builder()  
  .name("architect_agent")  
  .model(chatModel)  
  .instruction(instruction)  
  .build();
```

### 使用流式输出 (Streaming)[​](#使用流式输出-streaming "使用流式输出 (Streaming)的直接链接")

Spring AI Alibaba 的 ReactAgent 提供了对流式输出的原生支持，基于 Reactor Flux 实现。这使得 Agent 能够实时推送推理过程、工具调用状态和最终响应。

#### 核心代码示例[​](#核心代码示例 "核心代码示例的直接链接")

```
import com.alibaba.cloud.ai.graph.agent.ReactAgent;  
import com.alibaba.cloud.ai.graph.agent.OutputType;  
import org.springframework.ai.chat.messages.AssistantMessage;  
import org.springframework.ai.chat.messages.ToolResponseMessage;  
import reactor.core.publisher.Flux;  
  
// 1. 获取流式响应  
Flux<Message> flux = agent.stream("查询杭州明天的天气");  
  
// 2. 订阅处理  
flux.subscribe(message -> {  
    // 处理各种类型的消息  
    handleMessage(message);  
});  
  
// 消息处理逻辑  
void handleMessage(Message message) {  
  OutputType type = (OutputType) message.getMetadata().get("type");  
  
  // 处理模型生成的 Token  
  if (type == OutputType.AGENT_MODEL_TOKEN) {  
      if (message instanceof AssistantMessage assistantMessage) {  
          System.out.print(assistantMessage.getText());  
       }  
  }  
  // 处理思考过程 (Reasoning) - 针对支持思考的模型如 DeepSeek-R1  
  else if (type == OutputType.AGENT_THINK_TOKEN) {  
       if (message instanceof AssistantMessage assistantMessage) {  
           String reasoning = (String) assistantMessage.getMetadata().get("reasoningContent");  
           if (reasoning != null) {  
               System.out.print("[Thinking] " + reasoning);  
           }  
       }  
  }  
  // 处理模型输出完成  
  else if (type == OutputType.AGENT_MODEL_FINISHED) {  
      if (message instanceof AssistantMessage assistantMessage) {  
          if (assistantMessage.hasToolCalls()) {  
              // 工具调用请求  
              assistantMessage.getToolCalls().forEach(toolCall -> {  
                  System.out.println("[Tool Call] " + toolCall.name() + ": " + toolCall.arguments());  
              });  
          } else {  
              // 模型完整响应  
              System.out.println("\n[Model Finished]");  
          }  
      }  
  }  
  // 处理工具执行结果  
  else if (type == OutputType.AGENT_TOOL_FINISHED) {  
      if (message instanceof ToolResponseMessage toolResponse) {  
          toolResponse.getResponses().forEach(response -> {  
              System.out.println("[Tool Result] " + response.name() + ": " + response.responseData());  
          });  
      }  
  }  
}
```

提示

* **先判断 OutputType**：通过 `OutputType` 先确定是模型输出、工具输出还是 Hook 输出，再处理具体消息
* **Thinking 消息**：部分模型（如 DeepSeek-R1）支持输出思考过程，通过 `metadata.reasoningContent` 获取
* **工具调用**：工具调用请求通常在 `AGENT_MODEL_FINISHED` 阶段出现，此时 `hasToolCalls()` 返回 `true`
* **流式 vs 完成**：`STREAMING` 阶段内容是增量的，`FINISHED` 阶段可获取完整消息或执行结果

更多关于 Graph 底层流式输出机制，请参考 [Graph 流式输出](/docs/frameworks/graph-core/core/streaming)。

## 下一步[​](#下一步 "下一步的直接链接")

* 学习 [多 Agent 编排](/docs/frameworks/agent-framework/advanced/multi-agent) 构建复杂系统
* 探索 [Graph API](/docs/frameworks/agent-framework/advanced/workflow) 实现自定义工作流
* 查看 [工具开发](/docs/frameworks/agent-framework/tutorials/tools) 扩展 Agent 能力
* 参考 [示例项目](https://github.com/spring-ai-alibaba/examples) 获取实践指导
