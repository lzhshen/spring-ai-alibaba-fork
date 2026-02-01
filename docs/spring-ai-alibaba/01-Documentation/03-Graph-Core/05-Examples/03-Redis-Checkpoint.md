# Redis Checkpoint Persistence

Graph Core 提供了基于 Redis 的持久化实现，用于生产环境的状态保存，确保工作流状态可靠存储并支持恢复。

## 前置条件

-   Redis 6.0 或更高版本
-   Java 17+
-   Redisson 客户端

## 依赖配置

**Maven**

```xml
<dependency>
  <groupId>org.redisson</groupId>
  <artifactId>redisson</artifactId>
  <version>3.22.0</version>
</dependency>
```

**Gradle**

```gradle
implementation 'com.alibaba.cloud.ai:spring-ai-alibaba-graph-checkpoint-redis:1.0.0.3-SNAPSHOT'
implementation 'org.redisson:redisson:3.24.3'
```

## 配置与使用

配置 Redisson 客户端并创建 `RedisSaver`：

```java
Config config = new Config();
config.useSingleServer().setAddress("redis://localhost:6379");
RedissonClient redisson = Redisson.create(config);
RedisSaver saver = new RedisSaver(redisson);
SaverConfig saverConfig = SaverConfig.builder().register(saver).build();
```

### 在编译时应用

在图编译时包含 `saverConfig`。

### 恢复机制

通过指定 `threadId`，系统可以从 Redis 中检索特定的检查点（Checkpoint），从而恢复之前的工作流状态。

要从特定的 checkpoint-id 恢复：

```java
RunnableConfig checkpointConfig = RunnableConfig.builder()
      .threadId("thread-id")
      .checkPointId("specific-checkpoint-id")
      .build();
workflow.invoke(Map.of(), checkpointConfig);
```

## Spring Boot 集成

在 YAML 中定义 `spring.redis.host` 和 `port`。然后在 `@Configuration` 类中定义 `RedisSaver` Bean 和 `RedissonClient` Bean 以自动设置。
