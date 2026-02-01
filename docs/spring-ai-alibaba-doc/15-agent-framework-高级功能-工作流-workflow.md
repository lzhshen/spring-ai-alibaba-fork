## 工作流（Workflow）

Graph 是 Agent Framework 的底层运行时。**我们建议开发者使用Agent Framework，但直接使用Graph API也是完全可行的。**
Graph 是一个低级工作流和多智能体编排框架，使开发者能够实现复杂的应用程序编排。

## Agent 编排的核心引擎[​](#agent-编排的核心引擎 "Agent 编排的核心引擎的直接链接")

Spring AI Alibaba Graph 是 Agent 编排背后的核心引擎，在底层，Spring AI Alibaba 框架会将 Agent 编排为 Graph，组成一个由节点串联而成的 DAG 图。

### Graph 引擎核心概念与定义[​](#graph-引擎核心概念与定义 "Graph 引擎核心概念与定义的直接链接")

Spring AI Alibaba Graph 有以下三个核心概念：

* **状态（State）**：定义了在 Node 与 Edge 之间传递的数据结构，是整个 Agent 上下文传递的核心载体，具体实现上是一个 `Map<String, Object>`。
* **节点（Node）**：Graph 中的每个 Node 是执行逻辑单元，接受当前 State 作为输入，执行某些操作（如调用 LLM 或自定义逻辑），并返回对 State 的更新。
* **边（Edge）**：定义 Node 间的控制流，可为固定连接，也可依据状态条件动态决定下一步执行路径，实现分支逻辑

![](/assets/images/graph-a30bb7e2cc8d814278984aa0cf6f2f68.png)

通过组合 Node 和 Edge，开发者可以创建复杂的循环工作流，随着时间的推移不断更新 State 状态。然而，真正的力量来自 Spring AI Alibaba 如何管理这种 State 状态。

简而言之：Node 完成工作，Edge 告诉下一步该做什么。

### Graph 引擎提供的 Low-level API[​](#graph-引擎提供的-low-level-api "Graph 引擎提供的 Low-level API的直接链接")

Spring AI Alibaba 同时提供了声明式的 Agentic API 与底层原子化的 Graph API，两种模式都对开发者开发，\*\*Agentic API vs Graph API \*\*应该怎么选？
<truncated 32297 bytes>
台集成[​](#与dify低代码平台集成 "与Dify低代码平台集成的直接链接")

使用 Spring AI Alibaba Admin 平台，可以实现 Dify DSL 到 Spring AI Alibaba 高代码工程的导出。

### 压测数据[​](#压测数据 "压测数据的直接链接")

#### 压测集群规格[​](#压测集群规格 "压测集群规格的直接链接")

1. Spring AI Alibaba 工程，独立部署的容器，保持默认线程池等配置参数，2个POD，POD 规格 2C4G
2. Dify 平台，官方部署方式，保持默认配置参数，每个组件都拉起2个POD，POD 规格 2C4G

#### 有效并发处理上限[​](#有效并发处理上限 "有效并发处理上限的直接链接")

* **压测方式：** 每个场景从 10 个 RPS（Request Per Second）开始，逐步提升，直到提升 RPS 值并不能带来 TPS 提升、成功率答复下降。
* **结论：** Dify 能处理的上限 RPS < 10；Spring AI Alibaba 能处理的上限 RPS 约 150。

Dify 压测截图：

![Dify DSL to Graph](/assets/images/dify-base-rps-6bc2021056a312ecd454973f1d651529.png)

Spring AI Alibaba 压测截图：

![Dify DSL to Graph](/assets/images/spring-ai-alibaba-base-rps-084035ab6bde58ae0b33f047b41e3097.png)

#### 极限场景下的吞吐量[​](#极限场景下的吞吐量 "极限场景下的吞吐量的直接链接")

* **压测方式：** 给集群远高于合理并发的压测请求量（测试场景为 1000 RPS），看集群的吞吐量、成功率变化。
* **结论：** Dify 在此场景下成功率小于 10%，平均 RT 接近 60s，大部分请求出现超时（响应大于 60s）；Spring AI Alibaba 成功率变化不大，维持 99% 以上，平均 RT 也在 18s 左右。

Dify 压测截图：

![Dify DSL to Graph](/assets/images/dify-extreme-rps-ca79a7dac480f71232fa1fbc119ab945.png)

Spring AI Alibaba 压测截图：

![Dify DSL to Graph](/assets/images/spring-ai-alibaba-extreme-rps-341188751904f9eac2c8a1bb45114a14.png)

## 相关资源[​](#相关资源 "相关资源的直接链接")

* Graph 框架文档
