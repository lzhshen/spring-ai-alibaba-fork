# 上下文工程 (Context Engineering)

上下文工程是构建可靠 Agent 的核心。它涉及以正确的格式向 LLM 提供正确的信息和工具，使其能够准确完成任务。

## 为什么 Agent 会失败？

大多数 Agent 失败不是因为 LLM 能力不足，而是因为缺乏“正确”的上下文。
典型的 Agent 循环包括：
1. **模型调用**: 使用 Prompt 和工具调用 LLM。
2. **工具执行**: 执行 LLM 请求的工具。

要构建可靠的 Agent，你需要精细控制循环中的每个步骤。

## 上下文类型

Spring AI Alibaba 将上下文分为三类：

### 1. 模型上下文 (Model Context)
控制单次模型调用中包含的内容（瞬态）。

- **系统提示 (System Prompt)**: 设置 Agent 的行为和角色。
  - *动态提示*: `StateAwarePromptInterceptor` 可以根据对话长度或状态动态调整 Prompt。
  - *个性化提示*: `PersonalizedPromptInterceptor` 可以从长期记忆中加载用户偏好。
  
- **消息历史 (Messages)**: 控制发送给 LLM 的对话历史。
  - *消息过滤*: `MessageFilterInterceptor` 可以过滤掉不必要的历史消息，减少 Token 消耗。

- **工具 (Tools)**: 动态控制当前可用的工具。
  - *基于角色的工具*: 根据用户角色（Admin/User）动态决定 Agent 可以调用哪些工具。

- **模型选择**: 根据任务复杂度在不同模型（如 qwen-turbo vs qwen-max）之间动态切换。

### 2. 工具上下文 (Tool Context)
控制工具执行时可以访问和修改的状态。

工具可以通过 `ToolContext` 访问：
- **Agent 状态 (State)**: 读取或修改持久化状态。
- **配置 (Config)**: 获取运行时配置（如 userId）。

```java
// 工具访问状态示例
public class StatefulTool implements BiFunction<Request, ToolContext, Response> {
    public Response apply(Request request, ToolContext toolContext) {
        OverAllState state = (OverAllState) toolContext.getContext().get("state");
        // ...
    }
}
```

### 3. 生命周期上下文 (Lifecycle Context)
在 Agent 生命周期的特定阶段执行操作。

- **Hook 位置**: `BEFORE_AGENT`, `AFTER_AGENT`, `BEFORE_MODEL`, `AFTER_MODEL`.
- **用途**: 日志记录、状态检查、消息摘要等。

#### 消息摘要 Hook
这是一个常见的模式，用于防止上下文窗口溢出。
`SummarizationHook` (继承自 `MessagesModelHook`) 可以在消息数量超过阈值时，自动将旧消息压缩为摘要，并替换到上下文消息中。

```java
@HookPositions({HookPosition.BEFORE_MODEL})
public class SummarizationHook extends MessagesModelHook {
    // ...逻辑：如果消息过长，调用 LLM 生成摘要，并替换旧消息
}
```

## 最佳实践

- **区分瞬态与持久上下文**: `ModelInterceptor` 用于临时修改（瞬态），`ModelHook` 用于永久更新状态（持久）。
- **自动化上下文管理**: 利用 Interceptor 自动注入用户偏好、权限控制等，让 Agent 逻辑更纯粹。
- **积极管理 Token**: 使用过滤和摘要机制，避免上下文无限增长导致性能下降或成本增加。
