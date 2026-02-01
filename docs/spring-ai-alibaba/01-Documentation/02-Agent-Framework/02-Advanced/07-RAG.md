# 检索增强生成 (RAG)

RAG 技术允许 LLM 在运行时访问外部非结构化知识（如 PDF、数据库、网页等），从而提供更准确、基于事实的回答。

## RAG 架构

Spring AI Alibaba 支持三种主要的 RAG 架构：

### 1. 两步 RAG (2-step RAG)
最简单的模式，在生成之前先执行一次检索。
- **优点**: 简单、可预测。
- **实现**: 使用 `QuestionAnswerAdvisor` 或 `RetrievalAugmentationAdvisor`（开箱即用），或通过 `MessagesModelHook` 手动注入。

### 2. 智能体 RAG (Agentic RAG)
Agent 将检索作为工具（Tool Calling）使用，根据需要决策何时、如何检索。
- **优点**: 适应性强，支持多步检索和多源检索。
- **实现**: 将 `VectorStore` 包装为工具提供给 `ReactAgent`。

### 3. 混合 RAG (Hybrid RAG)
结合了上述两者，包含查询增强、检索验证和生成后检查等中间步骤。

## 核心组件

Spring AI Alibaba 提供了完整的 RAG 工具链（ETL Pipeline）：

1. **Document Readers**: 加载数据（PDF, Word, GitHub, Notion 等）。
2. **Document Transformers**: 文档转换（如文本分割 `TokenTextSplitter`）。
3. **Embedding Model**: 将文本转化为向量（DashScope, OpenAI 等）。
4. **Vector Store**: 向量存储（Milvus, Redis, Elasticsearch 等）。
5. **Retrievers**: 检索器接口，支持查询转换和后处理。

## 实现示例 (MessagesModelHook)

```java
class RAGMessagesHook extends MessagesModelHook {
    @Override
    public AgentCommand beforeModel(List<Message> previousMessages, RunnableConfig config) {
        // 1. 相似度搜索
        List<Document> docs = vectorStore.similaritySearch(userQuestion);
        // 2. 构造上下文
        String context = docs.stream().map(Document::getText).collect(Collectors.joining(""));
        // 3. 注入系统提示语
        enhancedMessages.add(new SystemMessage("基于以下上下文回答问题: " + context));
        return new AgentCommand(enhancedMessages, UpdatePolicy.REPLACE);
    }
}
```

## 最佳实践

- **选择合适架构**: FAQ 类用两步 RAG，复杂研究类用 Agentic RAG。
- **优化检索**: 使用合适的分割策略和重写（Rewrite）技术。
- **控制上下文**: 限制检索数量，防止超出模型 Context Window。
- **监控评估**: 跟踪检索准确率和回答的幻觉情况。
