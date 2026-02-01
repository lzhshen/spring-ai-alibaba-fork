# Agent Chat UI

Spring AI Alibaba Studio 提供了一个可视化的聊天界面，用于与您的 Agent 进行交互和调试。

## 快速体验

1.  **下载示例代码**:
    ```bash
    git clone https://github.com/alibaba/spring-ai-alibaba.git
    cd examples/deepresearch
    ```

2.  **启动 Agent 后端**:
    ```bash
    export AI_DASHSCOPE_API_KEY=your_key
    mvn spring-boot:run
    ```

3.  **访问 UI**:
    打开 `http://localhost:3000` 即可开始对话。

## 工作模式

Agent Chat UI 支持两种工作模式：

### 1. 嵌入式模式 (Embedded Mode)

将 UI 直接集成到您的 Spring Boot 应用中。

**依赖配置**:
```xml
<dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-studio</artifactId>
    <version>1.1.0.0-RC2</version>
</dependency>
```

**访问方式**:
启动应用后，访问 `http://localhost:{port}/chatui/index.html`。

### 2. 独立模式 (Standalone Mode)

作为独立的前端应用运行（基于 Next.js）。

**安装与运行**:
```bash
git clone https://github.com/alibaba/spring-ai-alibaba.git
cd spring-ai-alibaba/spring-ai-alibaba-studio/agent-chat-ui

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev
```

**配置**:
默认连接到 `http://localhost:8080`。你可以在 `.env.development` 文件中修改后端地址：

```env
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXT_PUBLIC_APP_NAME=research_agent
NEXT_PUBLIC_USER_ID=user-001
```
