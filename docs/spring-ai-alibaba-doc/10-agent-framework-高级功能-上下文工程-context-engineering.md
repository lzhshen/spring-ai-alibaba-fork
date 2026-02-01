## 上下文工程（Context Engineering）

## 概述[​](#概述 "概述的直接链接")

构建 Agent 的难点在于使其足够可靠、效果足够好。虽然我们可以很容易写一个 Agent 示例，但要做一个能在生产环境中稳定使用、能解决实际问题的 Agent 并不容易。

### 为什么 Agent 会失败？[​](#为什么-agent-会失败 "为什么 Agent 会失败？的直接链接")

当 Agent 失败时，通常是因为 Agent 内部的 LLM 调用采取了错误的操作或者没有按我们预期的执行。LLM 失败的原因有两个：

1. 底层 LLM 能力不足
2. 没有向 LLM 传递"正确"的上下文

大多数情况下 —— 实际上是第二个原因导致 Agent 不可靠。

**上下文工程**是以正确的格式提供正确的信息和工具，使 LLM 能够完成任务。这是 AI 工程师的首要工作。缺乏"正确"的上下文是更可靠 Agent 的头号障碍，Spring AI Alibaba 的 Agent 抽象专门设计用于优化上下文工程。

### Agent 循环[​](#agent-循环 "Agent 循环的直接链接")

典型的 Agent 循环由两个主要步骤组成：

1. **模型调用** - 使用提示和可用工具调用 LLM，返回响应或执行工具的请求
2. **工具执行** - 执行 LLM 请求的工具，返回工具结果

![reactagent](/assets/images/reactagent-62b03d114223a5a926dbc975846eb405.png)

此循环持续进行，直到 LLM 决定任务完成并退出。

### 你可以控制什么[​](#你可以控制什么 "你可以控制什么的直接链接")

要构建可靠的 Agent，你需要控制 Agent 循环每个步骤发生的事情，以及步骤之间发生的事情。

| 上下文类型 | 你控制的内容 | 瞬态或持久 |
| --- | --- | --- |
| **[模型上下文](#model-context)** | 模型调用中包含什么（指令、消息历史、工具、响应格式） | 瞬态 |
| **[工具上下文](#tool-context)** | 工具可以访问和产生
<truncated 20140 bytes>
话摘要：" + summary  
          );  
  
          // 保留最近的几条消息  
          int recentCount = Math.min(5, previousMessages.size());  
          List<Message> recentMessages = previousMessages.subList(  
              previousMessages.size() - recentCount,  
              previousMessages.size()  
          );  
  
          // 构建新的消息列表  
          List<Message> newMessages = new ArrayList<>();  
            
          // 保留原有的 SystemMessage（如果存在）  
          if (existingSystemMessage != null) {  
              newMessages.add(existingSystemMessage);  
          }  
            
          // 添加摘要上下文消息  
          newMessages.add(summaryContextMessage);  
            
          // 添加最近的消息，排除旧的 SystemMessage（如果存在）  
          for (Message msg : recentMessages) {  
              if (msg != existingSystemMessage) {  
                  newMessages.add(msg);  
              }  
          }  
  
          // 使用 REPLACE 策略替换消息列表  
          return new AgentCommand(newMessages, UpdatePolicy.REPLACE);  
      }  
  
      // 如果消息数量未超过阈值，返回原始消息（不进行修改）  
      return new AgentCommand(previousMessages);  
  }  
  
  private String generateSummary(List<Message> messages) {  
      // 使用另一个模型生成摘要  
      String conversation = messages.stream()  
          .map(Message::getText)  
          .collect(Collectors.joining("  
"));  
  
      // 简化示例：返回固定摘要  
      return "之前讨论了多个主题...";  
  }  
}
```

## 相关文档[​](#相关文档 "相关文档的直接链接")

* [Hooks](/docs/frameworks/agent-framework/tutorials/hooks) - Hook 机制详解
* [Interceptors](/docs/frameworks/agent-framework/tutorials/hooks) - 拦截器详解
* [Agents](/docs/frameworks/agent-framework/tutorials/agents) - Agent 基础概念
* [Memory](/docs/frameworks/agent-framework/advanced/memory) - 状态和记忆管理
