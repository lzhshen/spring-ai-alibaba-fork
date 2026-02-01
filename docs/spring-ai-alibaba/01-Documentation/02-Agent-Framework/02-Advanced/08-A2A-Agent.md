# 分布式智能体 (A2A Agent)

Agent2Agent (A2A) 协议旨在解决不同框架、不同位置部署的智能体之间的通信与协作问题。

## A2A 架构

Spring AI Alibaba 的 A2A 实现包含三个核心组件：
1. **A2A Server**: 将本地 Agent 暴露为标准的 A2A 服务。
2. **A2A Registry**: 智能体注册中心（支持 Nacos）。
3. **A2A Discovery**: 智能体发现机制（支持 Nacos）。

## 发布 A2A 智能体

### 1. 定义本地 Agent
定义一个普通的 `ReactAgent` 并作为 Bean 注册。

### 2. 配置 A2A Server
在 `application.yml` 中开启配置：

```yaml
spring:
  ai:
    alibaba:
      a2a:
        server:
          version: 1.0.0
          card:
            name: data_analysis_agent
            description: 专门用于数据分析的本地智能体
```

启动后，系统会自动暴露两个端点：
- `/.well-known/agent.json`: AgentCard 元数据。
- `/a2a/message`: 被调用的业务端点。

## 调用远程智能体

使用 `A2aRemoteAgent` 来代理远程服务。

```java
A2aRemoteAgent remote = A2aRemoteAgent.builder()
    .name("data_analysis_agent")
    .agentCardProvider(agentCardProvider) // 从 Nacos 发现
    .build();

// 像本地 Agent 一样调用
Optional<OverAllState> result = remote.invoke("分析上月销售数据");
```

## 工作流程 (配合 Nacos)

1. **Provider**: 应用启动，将 `AgentCard` 注册到 Nacos。
2. **Consumer**: 通过 `AgentCardProvider` 查询 Nacos 获取远程地址。
3. **Execution**: Consumer 发起 HTTP 调用到 Provider 的 `/a2a/message`。

## 注意事项

- **Bean 名称一致性**: `server.card.name` 必须匹配 Spring Context 中的 `ReactAgent` Bean 名称。
- **服务治理**: 依赖 `nacos-discovery` 模块进行节点健康检查。
- **跨组织协作**: A2A 协议不仅支持内部应用间通信，也为跨组织的 Agent 协作提供了标准。
