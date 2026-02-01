## Models 模型

## 概述[​](#概述 "概述的直接链接")

ChatModel API 为开发者提供了将 AI 驱动的聊天补全功能集成到应用程序中的能力。它利用预训练的语言模型（如 GPT），根据用户的自然语言输入生成类似人类的响应。

该 API 通常通过向 AI 模型发送提示或部分对话来工作，然后模型根据其训练数据和对自然语言模式的理解生成对话的完成或延续。完成的响应随后返回给应用程序，应用程序可以将其呈现给用户或用于进一步处理。

`Spring AI ChatModel API` 被设计为一个简单且可移植的接口，用于与各种 AI 模型交互，允许开发者在不同模型之间切换时只需最少的代码更改。

借助 `Prompt`（用于输入封装）和 `ChatResponse`（用于输出处理）等配套类，ChatModel API 统一了与 AI 模型的通信。它管理请求准备和响应解析的复杂性，提供直接且简化的 API 交互。

Spring AI ChatModel API 构建在 Spring AI `Generic Model API` 之上，提供 Chat 特定的抽象和实现。这允许轻松集成和在不同 AI 服务之间切换，同时为客户端应用程序维护一致的 API。

## Generic Model API[​](#generic-model-api "Generic Model API的直接链接")

为了为所有 AI Models 提供基础，Spring AI 创建了 Generic Model API。这使得通过遵循通用模式轻松地为 Spring AI 贡献新的 AI Model 支持。

以下类图展示了 Generic Model API 的架构：

![Spring AI Generic Model API](/assets/images/spring-ai-generic-model-api-2bb366b8599c94bec2f4b8d515d5c420.jpg)

### Model[​](#model "Model的直接链接")

`Model` 接口提供了调用 AI model 的通用 API。它旨在通过抽象发送请求和接收响应的过程来处理与各种类型的 AI model 的交互。该接口使用 Java 泛型来容纳不同类型的请求和响应，增强了不同 AI model 实现的灵活性和适应性
