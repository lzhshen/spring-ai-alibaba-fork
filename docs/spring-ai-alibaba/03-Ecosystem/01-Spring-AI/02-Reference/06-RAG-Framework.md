# RAG (Retrieval Augmented Generation) 框架

RAG 是克服大语言模型幻觉、处理海量私有数据的核心技术。Spring AI Alibaba 提供了一套完整的 RAG 抽象。

## 核心流程
1. **数据摄入 (Ingestion)**: 
   - 读取文档（PDF, Word, Markdown 等）。
   - 文档切分 (Splitting)。
   - Embedding 向量化。
   - 存入向量数据库 (Vector Store)。
2. **检索 (Retrieval)**: 根据用户提问及其向量，匹配最相关的文档片段。
3. **增强 (Augmentation)**: 将召回的内容作为上下文嵌入到 Prompt 中。
4. **生成 (Generation)**: LLM 基于增强后的 Prompt 给出回答。

## 关键组件：Advisors
Spring AI 使用 `Advisor` 来无感地处理 RAG 逻辑：
- **QuestionAnswerAdvisor**: 极简封装，自动执行“检索 -> 构造 Prompt -> 调用模型”的全流程。
- **RetrievalAugmentationAdvisor**: 提供更细粒度的控制，支持自定义检索策略和后处理。

## 模块化设计
- **Pre-Retrieval**: 对问题进行改写或扩充，提高检索精度。
- **Retrieval**: 多路召回或混合搜索。
- **Post-Retrieval**: 对检索结果进行重排序 (Rerank) 或过滤。
