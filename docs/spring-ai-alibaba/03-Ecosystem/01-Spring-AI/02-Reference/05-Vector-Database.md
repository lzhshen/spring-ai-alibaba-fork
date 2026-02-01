# 向量数据库 (Vector Databases)

向量数据库是 RAG 系统中用于存储和检索 Embedding 的核心设施。

## 支持的数据库
Spring AI Alibaba 适配了多种主流向量数据库：
- **AnalyticDB (ADB)**: 阿里云原生的云原生数据库，高性能向量索引。
- **DashVector**: 阿里云自研的向量检索服务。
- **PostgreSQL (PGVector)**: 基于经典数据库的向量扩展。
- **Milvus / Pinecone / Redis**: 全球主流的专用或通用型向量存储。

## 核心配置
通过 Spring Boot 的 `application.yml` 即可快速开启集成（以 AnalyticDB 为例）：
```yaml
spring:
  ai:
    vectorstore:
      analyticdb:
        endpoint: ${ADB_ENDPOINT}
        instance-id: ${ADB_INSTANCE_ID}
        collection-name: ${COLLECTION}
```

## 功能特性
- **相似性搜索**: 支持欧式距离、余弦距离等多种搜索算法。
- **元数据过滤 (Metadata Filtering)**: 在向量搜索的同时，支持通过 SQL 风格的表达式过滤非向量字段。
- **自动映射**: 自动将 `Document` 对象与数据库表字段进行映射，无需手动编写 SQL。
