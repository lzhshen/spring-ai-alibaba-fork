# Graph 工作流编排指南

学习如何通过将客服邮件处理流程分解为离散步骤来使用 Spring AI Alibaba Graph 构建智能工作流。

Spring AI Alibaba Graph 可以改变您构建智能代理的思维方式。使用 Graph 构建代理时，您将首先把它分解为称为**节点（nodes）**的离散步骤。然后，描述每个节点的不同决策和转换。最后，通过一个共享的**状态（state）**将节点连接起来，每个节点都可以读取和写入该状态。在本教程中，我们将指导您完成使用 Spring AI Alibaba Graph 构建客服邮件处理代理的思维过程。

## 从需要自动化的流程开始

假设您需要构建一个处理客服邮件的 AI 代理。产品团队给出了以下需求：

代理应该：
- 读取客户邮件
- 按紧急程度和主题分类
- 搜索相关文档以回答问题
- 起草适当的回复
- 将复杂问题上报给人工客服
- 在需要时安排后续跟进

需要处理的示例场景：
- 简单产品问题："如何重置我的密码？"
- Bug 报告："导出功能在选择 PDF 格式时崩溃"
- 紧急账单问题："我的订阅被重复扣费了！"
- 功能请求："能在移动应用中添加暗黑模式吗？"
- 复杂技术问题："我们的 API 集成间歇性失败，返回 504 错误"

要在 Spring AI Alibaba Graph 中实现代理，通常遵循以下五个步骤。

## 步骤 1：将工作流映射为离散步骤

首先识别流程中的不同步骤。每个步骤将成为一个节点（执行特定任务的函数）。然后勾画这些步骤如何相互连接。

**工作流示意图：**
START -> 读取邮件 -> 分类意图 -> (文档搜索 / Bug跟踪 / 人工审核) -> 起草回复 -> (人工审核 / 发送回复) -> END

箭头显示可能的路径，但具体选择哪条路径的决策发生在每个节点内部。

现在您已经识别了工作流中的组件，让我们了解每个节点需要做什么：
- **读取邮件**：提取和解析邮件内容
- **分类意图**：使用 LLM 对紧急程度和主题进行分类，然后路由到适当的操作
- **文档搜索**：查询知识库以获取相关信息
- **Bug跟踪**：在跟踪系统中创建或更新问题
- **起草回复**：生成适当的响应
- **人工审核**：上报给人工客服进行审批或处理
- **发送回复**：发送邮件响应

> **提示**：注意有些节点会决定接下来去哪里（分类意图、起草回复、人工审核），而其他节点总是进入相同的下一步。

## 步骤 2：确定每个步骤需要做什么

对于图中的每个节点，确定它代表什么类型的操作以及它需要什么上下文才能正常工作。

### LLM 步骤
当步骤需要理解、分析、生成文本或做出推理决策时：
- **分类意图节点**：
  - 静态上下文（提示）：分类类别、紧急程度定义、响应格式
  - 动态上下文（来自状态）：邮件内容、发件人信息
  - 期望结果：确定路由的结构化分类
- **起草回复节点**：
  - 静态上下文（提示）：语气指南、公司政策、响应模板
  - 动态上下文（来自状态）：分类结果、搜索结果、客户历史
  - 期望结果：准备好审核的专业邮件响应

### 数据步骤
当步骤需要从外部源检索信息时：
- **文档搜索节点**：
  - 参数：从意图和主题构建的查询
  - 错误处理：捕获异常并存储错误信息
- **客户历史查询**：
  - 参数：来自状态的客户邮箱或 ID

### 操作步骤
当步骤需要执行外部操作时：
- **发送回复节点**：批准后执行。
- **Bug跟踪节点**：当意图是 "bug" 时执行。

### 用户输入步骤
- **人工审核节点**：在高紧急程度、复杂问题或质量问题时触发。

## 步骤 3：设计您的状态

状态是图中所有节点可访问的共享记忆。

### 包含在状态中的内容
- 原始邮件和发件人信息
- 分类结果
- 搜索结果和客户数据
- 草稿响应
- 执行元数据

### 定义状态和状态键策略
```java
import com.alibaba.cloud.ai.graph.KeyStrategy;
import com.alibaba.cloud.ai.graph.KeyStrategyFactory;
import com.alibaba.cloud.ai.graph.state.strategy.ReplaceStrategy;
import com.alibaba.cloud.ai.graph.state.strategy.AppendStrategy;
import java.util.Map;
import java.util.HashMap;

// 邮件分类结构
public static class EmailClassification {
    private String intent;      // "question", "bug", "billing", "feature", "complex"
    private String urgency;     // "low", "medium", "high", "critical"
    private String topic;
    private String summary;

    // Getters and Setters...
}

// 配置状态键策略
public static KeyStrategyFactory createKeyStrategyFactory() {
    return () -> {
        HashMap<String, KeyStrategy> strategies = new HashMap<>();
        strategies.put("email_content", new ReplaceStrategy());
        strategies.put("classification", new ReplaceStrategy());
        strategies.put("search_results", new ReplaceStrategy());
        strategies.put("draft_response", new ReplaceStrategy());
        strategies.put("messages", new AppendStrategy());
        // ... 其他策略
        return strategies;
    };
}
```

## 步骤 4：构建您的节点

### 实现节点示例
```java
// 分类意图节点
public static class ClassifyIntentNode implements NodeAction {
    private final ChatClient chatClient;

    public ClassifyIntentNode(ChatClient.Builder chatClientBuilder) {
        this.chatClient = chatClientBuilder.build();
    }

    @Override
    public Map<String, Object> apply(OverAllState state) throws Exception {
        String emailContent = state.value("email_content").map(v -> (String) v).orElseThrow();
        
        String classificationPrompt = String.format("分析邮件并分类: %s", emailContent);
        String response = chatClient.prompt().user(classificationPrompt).call().content();

        // 解析并决定 next_node
        EmailClassification classification = parseClassification(response);
        String nextNode = "draft_response"; // 简化逻辑
        
        return Map.of("classification", classification, "next_node", nextNode);
    }
}
```

## 步骤 5：组装 Graph

使用 `StateGraph` 将节点连接起来，并配置持久化和中断点。

```java
public static CompiledGraph createEmailAgentGraph(ChatModel chatModel) throws GraphStateException {
    StateGraph workflow = new StateGraph(createKeyStrategyFactory())
            .addNode("read_email", node_async(new ReadEmailNode()))
            .addNode("classify_intent", node_async(new ClassifyIntentNode(ChatClient.builder(chatModel))))
            // ... 添加其他节点
            ;

    workflow.addEdge(START, "read_email");
    workflow.addEdge("read_email", "classify_intent");
    
    // 添加条件边
    workflow.addConditionalEdges("classify_intent",
            edge_async(state -> (String) state.value("next_node").orElse("draft_response")),
            Map.of("human_review", "human_review", "draft_response", "draft_response"));

    // 配置持久化和人工审核中断
    var compileConfig = CompileConfig.builder()
            .saverConfig(SaverConfig.builder().register(new MemorySaver()).build())
            .interruptBefore("human_review")
            .build();

    return workflow.compile(compileConfig);
}
```

## 测试您的代理

通过 `app.stream()` 运行图，在遇到 `interruptBefore` 时会暂停，人工批准后可使用 `updateState` 恢复执行。

## 总结

1. **分解为离散步骤**：每个节点只做好一件事。
2. **状态共享**：存储原始数据，按需格式化提示。
3. **人工输入一等公民**：通过 `interruptBefore` 轻松实现人工审查。
4. **显式控制流**：节点内部决定下一步去向，使流程清晰可追溯。
