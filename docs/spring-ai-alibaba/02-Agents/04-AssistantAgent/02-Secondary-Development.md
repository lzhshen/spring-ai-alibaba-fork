# Assistant Agent 二次开发指南

Assistant Agent 提供了灵活的扩展机制，允许开发者根据业务需求定制智能助手。

## 架构概览

系统由以下核心部分组成：
- **Core Engine**: 核心编排引擎。
- **Extension Modules**: 扩展模块。
- **Prompt Tools**: 提示词工具。

## 扩展点 (SPI)

开发者可以通过 SPI (Service Provider Interface) 扩展以下能力：

- **SearchProvider**: 自定义数据搜索源。
- **CodeactTool**: 扩展业务逻辑执行工具。
- **ReplyChannelDefinition**: 定义新的交互渠道（如钉钉、飞书）。

## 定制化开发

1.  **配置**: 主要通过 `application.yml` 进行配置，工具配置在 `mcp-servers.json` 中。
2.  **Prompt 优化**: 使用 `PromptBuilder` 组装特定场景的指令。
3.  **经验学习**: 利用 `LearningExtractor` 从历史执行中提取经验。
4.  **组件替换**: 通过定义带有 `@Primary` 注解的 Spring Bean 来覆盖默认组件。
