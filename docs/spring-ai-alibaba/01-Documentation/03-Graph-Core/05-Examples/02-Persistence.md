# 混合存储示例 (Persistence)

你可以为不同的 Namespace 配置不同的 Saver。

- **短期状态**: RedisSaver (快速、过期自动清理)。
- **长期审计**: PostgreSqlSaver (持久、可查询)。
