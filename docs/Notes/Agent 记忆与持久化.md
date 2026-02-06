---
tags:
  - Memory
  - Persistence
  - State
aliases:
  - Agent记忆
created: 2026-02-05
---

# Agent 记忆与持久化

## 核心理解 (My Understanding)
*   **无记忆状态**：“如果不配置 Memory Saver，模型目前应该是记不住上一句说了什么。”
*   **生产环境建议**：“如果将 Typeless Memory Saver 用于生产环境，最好是使用一个 Redis Saver 或者其他持久化的存储方式（比如 MongoDB 等数据库）。”

## 补充/修正 (Refinements)
*   *(注：MemorySaver 仅存储在 JVM 内存中，重启即失，且不支持多实例共享)*

## 官方定义与溯源 (Reference)
> [!quote] Source
> 原始文档：[[Agentic-Framework/01-Agents.md]]
>
> > [!info] Original Text
> > Agent 通过状态自动维护对话历史。使用 MemorySaver 配置持久化存储... 生产环境：使用 RedisSaver、MongoSaver 等持久化存储替代 MemorySaver。
