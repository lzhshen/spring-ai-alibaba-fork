## Messages 消息

Messages 是 Spring AI Alibaba 中模型交互的基本单元。它们代表模型的输入和输出，携带在与 LLM 交互时表示对话状态所需的内容和元数据。

Messages 是包含以下内容的对象：

* **Role（角色）** - 标识消息类型（如 `system`、`user`、`assistant`）
* **Content（内容）** - 表示消息的实际内容（如文本、图像、音频、文档等）
* **Metadata（元数据）** - 可选字段，如响应信息、消息 ID 和 token 使用情况

Spring AI Alibaba 提供了一个标准的消息类型系统，可在所有模型提供商之间工作，确保无论调用哪个模型都具有一致的行为。

## 基础使用[​](#基础使用 "基础使用的直接链接")

使用 messages 最简单的方式是创建消息对象并在调用模型时传递它们。

```
import org.springframework.ai.chat.model.ChatModel;  
import org.springframework.ai.chat.messages.UserMessage;  
import org.springframework.ai.chat.messages.SystemMessage;  
import org.springframework.ai.chat.messages.AssistantMessage;  
import org.springframework.ai.chat.prompt.Prompt;  
import java.util.List;  
  
// 使用 DashScope ChatModel  
ChatModel chatModel = // ... 初始化 ChatModel  
  
SystemMessage systemMsg = new SystemMessage("你是一个有帮助的助手。");  
UserMessage userMsg = new UserMessage("你好，你好吗？");  
  
// 与聊天模型一起使用  
List<org.springframework.ai.chat.messages.Message> messages = List.of(systemMsg, userMsg);  
Prompt prompt = new Prompt(messages);  
ChatResponse response = chatModel.call(prompt);  // 返回 ChatResponse，包含 AssistantMessage
```

### 文本提示[​](#文本提示 "文本提示的直接链接")

文本提示是字符串 - 适用于简单的生成任务，不需要保留对话历史。

```
// 使用字符串直接调用  
String response = chatModel.call("写一首关于春天的俳句");
```

**使用文
<truncated 12857 bytes>
r 模式[​](#使用-builder-模式 "使用 Builder 模式的直接链接")

Spring AI Alibaba 的消息类提供了 builder 模式以便于构建：

```
// UserMessage with builder  
UserMessage userMsg = UserMessage.builder()  
  .text("你好，我想学习 Spring AI Alibaba")  
  .metadata(Map.of("user_id", "user_123"))  
  .build();  
  
// SystemMessage with builder  
SystemMessage systemMsg = SystemMessage.builder()  
  .text("你是一个 Spring 框架专家")  
  .metadata(Map.of("version", "1.0"))  
  .build();  
  
// AssistantMessage with builder  
AssistantMessage assistantMsg = AssistantMessage.builder()  
  .content("我很乐意帮助你学习 Spring AI Alibaba！")  
  .build();
```

### 消息复制和修改[​](#消息复制和修改 "消息复制和修改的直接链接")

```
// 复制消息  
UserMessage original = new UserMessage("原始消息");  
UserMessage copy = original.copy();  
  
// 使用 mutate 创建修改的副本  
UserMessage modified = original.mutate()  
  .text("修改后的消息")  
  .metadata(Map.of("modified", true))  
  .build();
```

## 在 ReactAgent 中使用[​](#在-reactagent-中使用 "在 ReactAgent 中使用的直接链接")

ReactAgent 自动管理消息历史，但你也可以直接使用消息：

```
import com.alibaba.cloud.ai.graph.agent.ReactAgent;  
import org.springframework.ai.chat.messages.UserMessage;  
import org.springframework.ai.chat.messages.AssistantMessage;  
  
ReactAgent agent = ReactAgent.builder()  
  .name("my_agent")  
  .model(chatModel)  
  .systemPrompt("你是一个有帮助的助手")  
  .build();  
  
// 使用字符串  
AssistantMessage response1 = agent.call("你好");  
  
// 使用 UserMessage  
UserMessage userMsg = new UserMessage("帮我写一首诗");  
AssistantMessage response2 = agent.call(userMsg);  
  
// 使用消息列表  
List<Message> messages = List.of(  
  new UserMessage("我喜欢春天"),  
  new UserMessage("写一首关于春天的诗")  
);  
AssistantMessage response3 = agent.call(messages);
```
