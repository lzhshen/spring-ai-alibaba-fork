# Model Context Protocol (MCP)

**Model Context Protocol (MCP)** 是一个标准化协议，旨在让 AI 模型能够安全、统一地访问外部工具和资源。

## 架构

Spring AI MCP Java SDK 采用三层架构：

1.  **Client/Server Layer**:
    -   **Client**: 建立和管理与 MCP Server 的连接，处理协议版本协商、工具发现和执行。
    -   **Server**: 为 Client 提供工具、资源和功能，支持工具暴露、发现和基于 URI 的资源管理。
2.  **Session Layer (McpSession)**: 会话管理。
3.  **Transport Layer (McpTransport)**: 传输层。

## 集成方式

Spring AI Alibaba 通过 Spring Boot Starter 简化了集成：

-   **spring-ai-starter-mcp-client**: 用于连接 MCP 服务。
-   **spring-ai-starter-mcp-server**: 用于构建 MCP 服务。

支持的通信方式：
-   **Stdio**: 标准输入输出（适用于本地进程）。
-   **SSE (Server-Sent Events)**: 基于 HTTP，适用于网络通信。
