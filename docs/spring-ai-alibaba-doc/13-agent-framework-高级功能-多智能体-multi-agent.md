## 多智能体（Multi-agent）

**Multi-agent** 将复杂的应用程序分解为多个协同工作的专业化Agent。与依赖单个Agent处理所有步骤不同，**Multi-agent架构**允许你将更小、更专注的Agent组合成协调的工作流。

Multi-agent系统在以下情况下很有用：

* 单个Agent拥有太多工具，难以做出正确的工具选择决策
* 上下文或记忆增长过大，单个Agent难以有效跟踪
* 任务需要**专业化**（例如：规划器、研究员、数学专家）

## Multi-agent模式[​](#multi-agent模式 "Multi-agent模式的直接链接")

Spring AI Alibaba支持以下Multi-agent模式：

| 模式 | 工作原理 | 控制流 | 使用场景 |
| --- | --- | --- | --- |
| [**Tool Calling**](#tool-calling) | [Supervisor Agent将其他Agent作为*工具*调用](/docs/frameworks/agent-framework/advanced/agent-tool)。"工具"Agent不直接与用户对话——它们只执行任务并返回结果。 | 集中式：所有路由都通过调用Agent。 | 任务编排、结构化工作流。 |
| [**Handoffs**](#Handoffs) | 当前的Agent决定将控制权转移给另一个Agent。活动Agent随之变更，用户可以继续与新的Agent直接交互。 | 去中心化：Agent可以改变当前由谁来担当活跃Agent。 | 跨领域对话、专家接管。 |

## 选择模式[​](#选择模式 "选择模式的直接链接")

| 问题 | 工具调用 (Agent Tool) | 交接（Handoffs） |
| --- | --- | --- |
| 需要集中控制工作流程？ | ✅ 是 | ❌ 否 |
| 希望Agent直接与用户交互？ | ❌ 否 | ✅ 是 |
| 专家之间复杂的、类人对话？ | ❌ 有限 | ✅ 强 |

> 你可以混合使用两种模式——使用**交接**进行Agent切换，并让每个Agent**将子Agent作为工具调用**来执行专门任务。

关于工具调用模式的使用请查看 [Agent Tool 文档](/docs/frameworks/agent-framework/advanced/agent-tool)。

## 自定义Agent上
<truncated 42873 bytes>
ght]  
  .outputKey("analysis_result")  
  .build();  
  
// 3. 创建报告Agent（路由选择格式）  
ReactAgent pdfReportAgent = ReactAgent.builder()  
  .name("pdf_report")  
  .model(chatModel)  
  .description("生成PDF格式报告")  
  .instruction("""  
              请根据研究结果和分析结果生成一份PDF格式的报告。  
                
              研究结果：{research_data}  
              分析结果：{analysis_result}  
              """) // [!code highlight]  
  .outputKey("pdf_report")  
  .build();  
  
ReactAgent htmlReportAgent = ReactAgent.builder()  
  .name("html_report")  
  .model(chatModel)  
  .description("生成HTML格式报告")  
  .instruction("""  
              请根据研究结果和分析结果生成一份HTML格式的报告。  
                
              研究结果：{research_data}  
              分析结果：{analysis_result}  
              """) // [!code highlight]  
  .outputKey("html_report")  
  .build();  
  
LlmRoutingAgent reportAgent = LlmRoutingAgent.builder()  
  .name("report_router")  
  .description("根据需求选择报告格式")  
  .model(chatModel)  
  .subAgents(List.of(pdfReportAgent, htmlReportAgent))  
  .build();  
  
// 4. 组合成顺序工作流  
SequentialAgent hybridWorkflow = SequentialAgent.builder()  
  .name("research_workflow")  
  .description("完整的研究工作流：并行收集 -> 分析 -> 路由生成报告")  
  .subAgents(List.of(researchAgent, analysisAgent, reportAgent))  
  .build();  
  
// 使用  
Optional<OverAllState> result = hybridWorkflow.invoke("研究AI技术趋势并生成HTML报告");
```

## 相关文档[​](#相关文档 "相关文档的直接链接")

* [Agents](/docs/frameworks/agent-framework/tutorials/agents) - Agent基础概念
* [Tools](/docs/frameworks/agent-framework/tutorials/tools) - 工具的创建和使用
* [Hooks](/docs/frameworks/agent-framework/tutorials/hooks) - Hook机制
* [Memory](/docs/frameworks/agent-framework/advanced/memory) - 状态和记忆管理
