# Chat Client API (Ecosystem)

`ChatClient` 是 Spring AI Alibaba 提供的高级抽象接口，旨在以流式、类型安全的方式与各种 AI 模型进行交互。

## 核心功能
- **流式调用**: 支持 `call()` (阻塞同步) 和 `stream()` (反应式流)。
- **Prompt 模板支持**: 内置对提示词模板的处理，支持变量动态替换。
- **结构化输出**: 通过 `.entity()` 方法，直接将 AI 的响应映射为普通的 Java 对象（POJO），无需手动解析 JSON。
- **配置式开发**: 支持通过 `ChatClient.Builder` 进行自动配置，简化多模型管理。

## 代码示例
```java
// 获取返回实体
Person person = chatClient.prompt()
    .user("告诉我张三的信息")
    .call()
    .entity(Person.class);

// 使用模板
chatClient.prompt()
    .user(u -> u.text("你好，我是{name}").param("name", "李四"))
    .call();
```

## Advisors 机制
`ChatClient` 支持挂载各种 `Advisor` 来增强能力：
- **LoggingAdvisor**: 自动记录输入输出日志。
- **ChatMemoryAdvisor**: 自动处理多轮对话的上下文记忆。
- **QuestionAnswerAdvisor**: 快速实现 RAG 功能。
