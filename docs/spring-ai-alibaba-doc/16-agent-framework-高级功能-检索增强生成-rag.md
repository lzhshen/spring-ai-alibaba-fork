## 检索增强生成（RAG）

大型语言模型（LLM）虽然强大，但有两个关键限制：

* **有限的上下文**——它们无法一次性摄取整个语料库
* **静态知识**——它们的训练数据在某个时间点被冻结

检索通过在查询时获取相关的外部知识来解决这些问题。这是\*\*检索增强生成（RAG）\*\*的基础：使用特定上下文的信息来增强 LLM 的回答。

## 构建知识库[​](#构建知识库 "构建知识库的直接链接")

**知识库**是用于检索的文档或结构化数据的存储库。

如果你需要自定义知识库，可以使用 Spring AI Alibaba 的文档加载器和向量存储从你自己的数据构建。

> 如果你已经有一个知识库（例如 SQL 数据库、CRM 或内部文档系统），你**不需要**重建它。你可以：
>
> * 将其连接为 Agent 的**工具**用于 Agentic RAG
> * 查询它并将检索到的内容作为上下文提供给 LLM（[两步 RAG](#2-step-rag)）

### 从检索到 RAG[​](#从检索到-rag "从检索到 RAG的直接链接")

检索允许 LLM 在运行时访问相关上下文。但大多数实际应用更进一步：它们**将检索与生成集成**以产生基于事实的、上下文感知的答案。

这是\*\*检索增强生成（RAG）\*\*的核心思想。检索管道成为结合搜索和生成的更广泛系统的基础。

### 检索流程[​](#检索流程 "检索流程的直接链接")

典型的检索工作流如下：

![Spring AI Alibaba RAG](/assets/images/rag1-796f8a18e2c5786a4a3ea99580521510.png)

每个组件都是模块化的：你可以交换加载器、分割器、嵌入或向量存储，而无需重写应用程序的逻辑。

### 构建模块[​](#构建模块 "构建模块的直接链接")

在 Spring AI Alibaba 中，你可以使用以下组件构建 RAG 系统：

#### 文档加载器和解析器[​](#文档加载器和解析器 "文档加载器和解析器的直
<truncated 39713 bytes>
.client.ChatClient;  
import com.alibaba.cloud.ai.graph.agent.ReactAgent;  
  
// 工具（用于 Agentic RAG）  
import org.springframework.ai.tool.ToolCallback;  
import org.springframework.ai.tool.function.FunctionToolCallback;
```

### 模块化 RAG 架构[​](#模块化-rag-架构 "模块化 RAG 架构的直接链接")

Spring AI 实现了模块化 RAG 架构，支持灵活的组件组合：

* **Pre-Retrieval（检索前）**：查询转换（重写、压缩、翻译）、查询扩展（多查询扩展）
* **Retrieval（检索）**：文档搜索、文档连接
* **Post-Retrieval（检索后）**：文档后处理（重排序、去重、压缩）
* **Generation（生成）**：查询增强、上下文注入

这种模块化设计允许你根据需求灵活组合不同的组件，构建适合特定场景的 RAG 流程。详细说明请参考 [Retrieval Augmented Generation 模块文档](about:/integration/rag/retrieval-augmented-generation#modules)。

## 相关文档[​](#相关文档 "相关文档的直接链接")

### Agent Framework[​](#agent-framework "Agent Framework的直接链接")

* [Tools](/docs/frameworks/agent-framework/tutorials/tools) - 创建检索工具
* [Agents](/docs/frameworks/agent-framework/tutorials/agents) - 构建 Agentic RAG
* [Memory](/docs/frameworks/agent-framework/advanced/memory) - 对话记忆管理
* [Multi-Agent](/docs/frameworks/agent-framework/advanced/multi-agent) - 多 Agent 协作

### RAG 组件[​](#rag-组件 "RAG 组件的直接链接")

* [Retrieval Augmented Generation](/integration/rag/retrieval-augmented-generation) - RAG API 和模块化架构
* [ETL Pipeline](/integration/rag/etl-pipeline) - 数据提取、转换和加载
* [Document Readers](/integration/rag/document-readers) - 文档加载器实现
* [Document Parsers](/integration/rag/document-parsers) - 文档解析器实现
* [Embeddings](/integration/rag/embeddings) - 嵌入模型 API
* [Vector Databases](https://docs.spring.io/spring-ai/reference/api/vectordbs.html) - 向量数据库集成
