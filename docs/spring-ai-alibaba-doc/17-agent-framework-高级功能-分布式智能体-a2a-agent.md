## 分布式智能体（A2A Agent）

## A2A 协议简介[​](#a2a-协议简介 "A2A 协议简介的直接链接")

随着智能体应用的广泛落地，智能体应用间的分布式部署与远程通信成为要解决的关键问题，Google 推出的 [Agent2Agent（A2A）协议](https://a2a-protocol.org/latest/)即面向这一落地场景：A2A 解决智能体与其他使用不同框架、部署在不同机器、不同公司的智能体进行有效通信和协作的问题。

A2A 协议定义了智能体之间通信的标准方式，使得不同框架、不同部署环境的智能体能够无缝协作。

## A2A 架构[​](#a2a-架构 "A2A 架构的直接链接")

Spring AI Alibaba 的 A2A 实现包含三个核心组件：

1. **A2A Server**：将本地 ReactAgent 暴露为 A2A 服务
2. **A2A Registry**：Agent 注册中心（支持 Nacos）
3. **A2A Discovery**：Agent 发现机制（支持 Nacos）

### 注册与发现流程[​](#注册与发现流程 "注册与发现流程的直接链接")

```
┌─────────────────┐         ┌──────────────┐         ┌─────────────────┐  
│  Agent Provider │         │  Nacos       │         │  Agent Consumer │  
│  (本地Agent)    │────────▶│  Registry    │◀────────│  (远程调用)     │  
└─────────────────┘         └──────────────┘         └─────────────────┘  
       │                            │                          │  
       │ 1. 注册 AgentCard          │                          │  
       │───────────────────────────▶│                          │  
       │                            │                          │  
       │      
<truncated 8357 bytes>

3. 查看 A2A 服务注册维度，应该能看到注册的 Agent

### 注册与发现的区别[​](#注册与发现的区别 "注册与发现的区别的直接链接")

| 功能 | 配置项 | 作用 | 角色 |
| --- | --- | --- | --- |
| **Registry（注册）** | `registry.enabled: true` | 将本地 Agent 注册到 Nacos | 服务提供者 |
| **Discovery（发现）** | `discovery.enabled: true` | 从 Nacos 查询其他 Agent | 服务消费者 |

两者可独立配置，也可同时启用。当同时启用时：

* 作为**提供者**：注册本地 Agent 到 Nacos
* 作为**消费者**：可发现并调用其他已注册的 Agent（包括自己）

## 注意事项[​](#注意事项 "注意事项的直接链接")

1. **依赖要求**：

   * 需要添加 `spring-ai-alibaba-starter-a2a-nacos` 依赖
   * 确保 Nacos 服务正常运行
2. **AgentCard 元数据**：

   * `server.card.name` 必须与 ReactAgent Bean 的 `name` 一致
   * `server.card.provider` 可选，用于标识 Agent 提供者信息
3. **多 Agent 注册**：

   * 默认情况下，只有一个 Agent Bean 会被注册
   * 如需注册多个 Agent，需运行多个应用实例，每个实例配置不同的 Agent

## 故障排查[​](#故障排查 "故障排查的直接链接")

### Agent 没有注册到 Nacos[​](#agent-没有注册到-nacos "Agent 没有注册到 Nacos的直接链接")

* 检查 `registry.enabled: true` 是否配置
* 查看应用日志，确认 Nacos Registry AutoConfiguration 是否生效
* 验证 Nacos 连接配置（server-addr、username、password）

### AgentCardProvider 无法发现 Agent[​](#agentcardprovider-无法发现-agent "AgentCardProvider 无法发现 Agent的直接链接")

* 检查 `discovery.enabled: true` 是否配置
* 确认 Agent 已成功注册到 Nacos
* 验证 agent name 是否匹配

### 远程调用失败[​](#远程调用失败 "远程调用失败的直接链接")

* 确认目标 Agent 的 REST API 端点可访问
* 检查网络连接和防火墙配置
* 查看 A2A 消息传输日志
