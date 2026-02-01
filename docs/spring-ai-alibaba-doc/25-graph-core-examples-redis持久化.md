## Redis 检查点持久化

> 在 Redis 数据库中持久化和管理您的 Spring AI Alibaba Graph 工作流状态，确保持久性

## 概述[​](#概述 "概述的直接链接")

Redis 检查点持久化是 Spring AI Alibaba Graph 生态系统的一个模块，它使得工作流状态能够可靠地存储在 Redis 数据库中。这使您基于 LLM 的应用程序在执行之间保持状态——确保工作流进度不会丢失，并且可以在任何时候恢复或分析。

主要特性包括：

* **基于 Redis 的持久化**：所有工作流状态都存储在 Redis 数据库中，可以在进程重启或系统故障后保存。
* **高性能存储**：利用 Redis 的内存存储特性，提供极快的读写性能。
* **自动过期管理**：支持 TTL（Time To Live）配置，自动清理过期的检查点数据。

## 功能特性[​](#功能特性 "功能特性的直接链接")

* **持久化状态**：持久化 Spring AI Alibaba Graph 工作流的整个状态，允许随时继续或恢复。
* **高性能访问**：利用 Redis 的内存存储和数据结构，提供毫秒级的访问速度。
* **灵活的配置**：支持单机、哨兵、集群等多种 Redis 部署模式。
* **无缝集成**：开箱即用地与 Spring AI Alibaba Graph 的状态管理和工作流 API 配合使用。

## 要求[​](#要求 "要求的直接链接")

* **Redis 数据库**：推荐版本 6.0 或更高。
* **Java 17+**
* **Spring AI Alibaba Graph 核心库**
* **Redisson 客户端**：用于与 Redis 交互

## 快速开始[​](#快速开始 "快速开始的直接链接")

### 添加依赖[​](#添加依赖 "添加依赖的直接链接")

在您的项目构建配置中添加以下内容：

**Maven**

```
<!-- Redisson 客户端依赖 -->  
<dependency>  
  <groupId>org.redisson</groupId>  
  <artifactId>redisson</artifactId>  
  <version>3.22.0</version>  
</dependency>
```

**Gradle**

```
implementation 'com.alibaba
<truncated 10055 bytes>
除的直接链接")

### 连接问题[​](#连接问题 "连接问题的直接链接")

确保 Redis 服务器可访问且连接参数正确：

```
// 测试 Redis 连接  
try {  
  Config config = new Config();  
  config.useSingleServer()  
          .setAddress("redis://localhost:6379");  
    
  RedissonClient redisson = Redisson.create(config);  
    
  // 执行简单的 ping 操作测试连接  
  redisson.getKeys().count();  
  System.out.println("Redis 连接成功！");  
    
  redisson.shutdown();  
} catch (Exception e) {  
  System.err.println("Redis 连接失败: " + e.getMessage());  
}
```

### 内存问题[​](#内存问题 "内存问题的直接链接")

如果遇到内存不足的错误：

* 检查 Redis 内存使用情况：`redis-cli info memory`
* 配置 Redis 最大内存限制
* 为检查点数据设置合理的 TTL
* 考虑使用 Redis 集群模式分散内存压力

## 最佳实践[​](#最佳实践 "最佳实践的直接链接")

1. **唯一的线程 ID**：为每个独立的工作流实例使用唯一的线程 ID。
2. **定期备份**：配置 Redis 持久化（RDB 或 AOF）以防止数据丢失。
3. **监控**：监控 Redis 内存使用、连接数和性能指标。
4. **TTL 策略**：为检查点数据设置合理的过期时间，自动清理旧数据。
5. **安全性**：使用 Redis 密码认证，限制网络访问，启用 TLS 加密。
6. **高可用**：生产环境建议使用 Redis Sentinel 或 Cluster 模式。
7. **内存管理**：合理配置 Redis 最大内存和淘汰策略。

## 总结[​](#总结 "总结的直接链接")

Redis 检查点持久化为 Spring AI Alibaba Graph 提供了高性能的状态管理解决方案，使您的 AI 应用程序能够在故障后恢复，实现长时间运行的工作流，并支持人在回路中的交互。通过利用 Redis 的内存存储特性和高性能，您可以构建健壮的、生产级的 AI 应用程序。Redis 的快速读写能力和灵活的数据结构使其成为检查点持久化的理想选择。
