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

Agent 的**记忆 (Memory)** 实际上是对对话状态（State）的维护。

## Saver 机制
框架通过 `Saver` 接口抽象了状态存储：
1.  **MemorySaver**：
    *   存储在 JVM 堆内存中。
    *   **特点**：速度快，但重启即失，不支持多实例共享。
    *   **场景**：开发调试、本地测试。
2.  **持久化 Saver** (如 RedisSaver, JdbcSaver)：
    *   存储在外部数据库或缓存中。
    *   **特点**：数据持久化，支持集群部署。
    *   **场景**：**生产环境必选**。

若不配置 Saver，Agent 将无法在多轮对话中保持上下文（即"金鱼记忆"）。

## 关联笔记
- [[Spring AI Alibaba Graph 运行时架构]]
