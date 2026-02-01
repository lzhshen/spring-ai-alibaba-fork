## Memory 短期记忆

## 概述[​](#概述 "概述的直接链接")

记忆可以让 Agent 记住之前的会话内容。对于 AI Agent，记忆至关重要，因为它让它们能够记住先前的交互、从反馈中学习并适应用户偏好。随着 Agent 处理更复杂的任务和大量用户交互，这种能力对于效率和用户满意度都变得至关重要。

短期记忆让你的应用程序能够在单个线程或会话中记住先前的交互。

> **注意**：会话可以隔离同一个 Agent 实例中的多个不同交互，类似于电子邮件在单个对话中分组消息的方式。

## 理解 ReactAgent 中的短期记忆[​](#理解-reactagent-中的短期记忆 "理解 ReactAgent 中的短期记忆的直接链接")

Spring AI Alibaba 将短期记忆作为 Agent 状态的一部分进行管理。

通过将这些存储在 Graph 的状态中，Agent 可以访问给定对话的完整上下文，同时保持不同对话之间的分离。状态使用 checkpointer 持久化到数据库（或内存），以便可以随时恢复线程。短期记忆在调用 Agent 或完成步骤（如工具调用）时更新，并在每个步骤开始时读取状态。

## 记忆带来的上下文过长问题[​](#记忆带来的上下文过长问题 "记忆带来的上下文过长问题的直接链接")

保留所有对话历史是实现短期记忆最常见的形式。但较长的对话对历史可能会导致大模型 LLM 上下文窗口超限，导致上下文丢失或报错。

即使你在使用的大模型上下文长度足够大，大多数模型在处理较长上下文时的表现仍然很差。因为很多模型会被过时或偏离主题的内容"分散注意力"。同时，过长的上下文，还会带来响应时间变长、Token 成本增加等问题。

在 Spring AI ALibaba 中，ReactAgent 使用 [messages](/docs/frameworks/agent-framework/tutorials/messages) 记录和传递上下文，其中包括指令、用户输入、模型响应和工具输出。

`ReactAgent` 内置支持多种记忆管理策略，或者你可以使用 hooks 自定义。

## 如何使用短期记忆[​](#如何使用短期记忆 "如何使用短期记忆的直接链接")

`ReactAgent` 需要 `CheckpointSaver` 来持久化记忆。这允许您在多个回合中保持对话上下文。

**启用记忆的方法**：通过 `ReactAgent.builder().saver(...)` 配置一个 saver。

```
import com.alibaba.cloud.ai.graph.agent.memory.MemorySaver;  
  
// 使用内存存储（仅用于测试）  
ReactAgent agent = ReactAgent.builder()  
    .name("my_agent")  
    .model(chatModel)  
    .saver(new MemorySaver())  
    .build();
```

**持久化记忆**：对于生产环境，应使用数据库 backed 的 Saver，如 Postgres、Redis 等。Spring AI Alibaba 提供了多种实现。

当启用 Saver 后，您可以传递一个 `threadId` 给 `agent.call()`，Agent 将自动加载该线程的历史记录：

```
// 第一轮对话  
String response1 = agent.call("我叫小明。", "thread-1");  
System.out.println(response1); // 你好，小明。  
  
// 第二轮对话 - Agent 记得名字  
String response2 = agent.call("我叫什么名字？", "thread-1");  
System.out.println(response2); // 你叫小明。
```

## 管理记忆策略（Memory Management Strategies）[​](#管理记忆策略memory-management-strategies "管理记忆策略（Memory Management Strategies）的直接链接")

为了解决上下文过长的问题，我们需要对记忆进行管理。ReactAgent 允许您通过 Hooks 介入消息处理流程。

### 消息修剪（Message Trimming）[​](#消息修剪message-trimming "消息修剪（Message Trimming）的直接链接")

使用 `MessagesModelHook` 可以在调用模型之前修改消息列表。例如，只保留最近的 N 条消息：

```
import com.alibaba.cloud.ai.graph.agent.hook.messages.MessagesModelHook;  
import com.alibaba.cloud.ai.graph.agent.hook.HookPosition;  
import com.alibaba.cloud.ai.graph.agent.hook.HookPositions;  
import com.alibaba.cloud.ai.graph.agent.hook.messages.AgentCommand;  
import com.alibaba.cloud.ai.graph.RunnableConfig;  
import org.springframework.ai.chat.messages.Message;  
import java.util.List;  
import java.util.ArrayList;  
  
@HookPositions({HookPosition.BEFORE_MODEL})  
public class WindowMemoryHook extends MessagesModelHook {  
    private final int windowSize;  
  
    public WindowMemoryHook(int windowSize) {  
        this.windowSize = windowSize;  
    }  
  
    @Override  
    public String getName() {  
        return "WindowMemory";  
    }  
  
    @Override  
    public AgentCommand beforeModel(List<Message> previousMessages, RunnableConfig config) {  
        int size = previousMessages.size();  
        if (size <= windowSize) {  
            return new AgentCommand(previousMessages);  
        }  
        // 保留系统消息（通常是第一条）和最近的 windowSize - 1 条消息  
        List<Message> trimmed = new ArrayList<>();  
        if (!previousMessages.isEmpty() && previousMessages.get(0).getMessageType().equals(MessageType.SYSTEM)) {  
             trimmed.add(previousMessages.get(0));  
        }  
        // 添加最近的消息  
        int start = Math.max(1, size - windowSize + (trimmed.isEmpty() ? 0 : 1));  
        trimmed.addAll(previousMessages.subList(start, size));  
  
        return new AgentCommand(trimmed);  
    }  
}
```

## 高级用法 - 消息过滤与修正[​](#高级用法---消息过滤与修正 "高级用法 - 消息过滤与修正的直接链接")

除了修剪，Hooks 还可以用于过滤敏感信息或修正即时输出。

```
import com.alibaba.cloud.ai.graph.agent.hook.messages.UpdatePolicy;  
import com.alibaba.cloud.ai.graph.RunnableConfig;  
import org.springframework.ai.chat.messages.AssistantMessage;  
import org.springframework.ai.chat.messages.Message;  
import java.util.ArrayList;  
import java.util.List;  
  
@HookPositions({HookPosition.AFTER_MODEL})  
public class ValidateResponseHook extends MessagesModelHook {  
  
  private static final List<String> STOP_WORDS =  
      List.of("password", "secret", "api_key");  
  
  @Override  
  public String getName() {  
      return "validate_response";  
  }  
  
  @Override  
  public AgentCommand afterModel(List<Message> previousMessages, RunnableConfig config) {  
      if (previousMessages.isEmpty()) {  
          return new AgentCommand(previousMessages);  
      }  
  
      Message lastMessage = previousMessages.get(previousMessages.size() - 1);  
      String content = lastMessage.getText();  
  
      // 检查是否包含敏感词  
      for (String stopWord : STOP_WORDS) {  
          if (content.toLowerCase().contains(stopWord)) {  
              // 移除包含敏感词的消息，替换为安全消息  
              List<Message> filtered = new ArrayList<>(  
                  previousMessages.subList(0, previousMessages.size() - 1)  
              );  
              filtered.add(new AssistantMessage("抱歉，我无法提供该信息。"));  
              return new AgentCommand(filtered, UpdatePolicy.REPLACE);  
          }  
      }  
  
      return new AgentCommand(previousMessages);  
  }  
}  
  
ReactAgent agent = ReactAgent.builder()  
  .name("secure_agent")  
  .model(chatModel)  
  .hooks(new ValidateResponseHook())  
  .saver(new MemorySaver())  
  .build();
```

## 相关资源[​](#相关资源 "相关资源的直接链接")

* [Agents 文档](/docs/frameworks/agent-framework/tutorials/agents) - 了解 ReactAgent 的核心概念
* [Hooks 和 Interceptors](/docs/frameworks/agent-framework/tutorials/hooks) - 了解如何扩展 Agent 功能
* [Messages 文档](/docs/frameworks/agent-framework/tutorials/messages) - 了解消息类型和使用
