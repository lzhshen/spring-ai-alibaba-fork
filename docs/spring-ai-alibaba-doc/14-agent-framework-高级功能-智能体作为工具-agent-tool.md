## 智能体作为工具（Agent Tool）

**Multi-agent** 将复杂的应用程序分解为多个协同工作的专业化Agent。与依赖单个Agent处理所有步骤不同，**Multi-agent架构**允许你将更小、更专注的Agent组合成协调的工作流。

Multi-agent系统在以下情况下很有用：

* 单个Agent拥有太多工具，难以做出正确的工具选择决策
* 上下文或记忆增长过大，单个Agent难以有效跟踪
* 任务需要**专业化**（例如：规划器、研究员、数学专家）

## Multi-agent模式[​](#multi-agent模式 "Multi-agent模式的直接链接")

Spring AI Alibaba支持以下Multi-agent模式：

| 模式 | 工作原理 | 控制流 | 使用场景 |
| --- | --- | --- | --- |
| [**Tool Calling**](#tool-calling) | Supervisor Agent将其他Agent作为*工具*调用。"工具"Agent不直接与用户对话——它们只执行任务并返回结果。 | 集中式：所有路由都通过调用Agent。 | 任务编排、结构化工作流。 |
| [**Handoffs**](#Handoffs) | 当前的Agent决定将控制权转移给另一个Agent。活动Agent随之变更，用户可以继续与新的Agent直接交互。 | 去中心化：Agent可以改变当前由谁来担当活跃Agent。 | 跨领域对话、专家接管。 |

## 选择模式[​](#选择模式 "选择模式的直接链接")

| 问题 | 工具调用 | 交接（Handoffs） |
| --- | --- | --- |
| 需要集中控制工作流程？ | ✅ 是 | ❌ 否 |
| 希望Agent直接与用户交互？ | ❌ 否 | ✅ 是 |
| 专家之间复杂的、类人对话？ | ❌ 有限 | ✅ 强 |

> 你可以混合使用两种模式——使用**交接**进行Agent切换，并让每个Agent**将子Agent作为工具调用**来执行专门任务。

## 自定义Agent上下文[​](#自定义agent上下文 "自定义Agent上下文的直接链接")

Multi-agent设计的核心是**上下文工程**——决定每个Agent看到什么信息。Spring AI Ali
<truncated 12882 bytes>
章和内容生成")  
  .instruction("你是一个专业作家，擅长各类文章创作。")  
  .build();  
  
// 创建翻译Agent  
ReactAgent translatorAgent = ReactAgent.builder()  
  .name("translator_agent")  
  .model(chatModel)  
  .description("专门负责文本翻译工作")  
  .instruction("你是一个专业翻译，能够准确翻译多种语言。")  
  .build();  
  
// 创建总结Agent  
ReactAgent summarizerAgent = ReactAgent.builder()  
  .name("summarizer_agent")  
  .model(chatModel)  
  .description("专门负责内容总结和提炼")  
  .instruction("你是一个内容总结专家，擅长提炼关键信息。")  
  .build();  
  
// 创建主Agent，集成多个工具  
ReactAgent multiToolAgent = ReactAgent.builder()  
  .name("multi_tool_coordinator")  
  .model(chatModel)  
  .instruction("你可以访问多个专业工具：写作、翻译和总结。" +  
          "根据用户需求选择合适的工具来完成任务。")  
  .tools(  
      AgentTool.getFunctionToolCallback(writerAgent),      // [!code highlight]  
      AgentTool.getFunctionToolCallback(translatorAgent),  // [!code highlight]  
      AgentTool.getFunctionToolCallback(summarizerAgent)   // [!code highlight]  
  )  
  .build();  
  
// 使用 - 主Agent会根据需求自动选择合适的工具  
Optional<OverAllState> result = multiToolAgent.invoke(  
  "请写一篇关于AI的文章，然后翻译成英文，最后给出摘要");
```

在这种模式中：

1. **专业化分工**：每个子Agent专注于特定领域（写作、翻译、总结等）
2. **灵活组合**：主Agent可以根据任务需求调用一个或多个工具
3. **智能路由**：主Agent根据工具的描述和用户需求，自动选择合适的工具
4. **顺序执行**：主Agent可以按顺序调用多个工具，实现复杂的工作流

> **提示**：为每个子Agent提供清晰、准确的 `description` 非常重要，这直接影响主Agent如何选择合适的工具。描述应该简洁地说明Agent的职责和能力。
