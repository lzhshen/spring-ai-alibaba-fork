# Chat Models 集成与对比

Spring AI Alibaba 支持对接多种国内外顶尖的生成式 AI 模型。

## 支持的模型列表
- **阿里云通义千问 (DashScope)**: 深度优化，支持 Qwen-Max/Plus/Turbo。
- **DeepSeek**: 高性价比的开源模型方案。
- **OpenAI**: 全球公认的基座模型。
- **其他**: 如智谱 (Zhipu), 零一万物 (Minimax) 等。

## 模型能力对比维度
官方文档通常从以下维度评估模型：
- **推理能力 (Reasoning)**: 处理逻辑任务的表现。
- **多模态 (Multimodal)**: 是否支持图片、音频输入。
- **函数调用 (Tool Use)**: 触发 Tool Calling 的准确率。
- **上下文长度 (Context Window)**: 支持的最大 Token 数。

## 最佳实践
- **统一抽象**: 使用 `ChatModel` 接口，可以在不修改业务代码的情况下通过配置文件切换底层供应商。
- **按需分配**: 对逻辑复杂的任务使用 `Qwen-Max`，对日常简单对话使用 `Qwen-Turbo` 以节省成本。
