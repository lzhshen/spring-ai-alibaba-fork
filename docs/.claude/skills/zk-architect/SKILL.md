---
name: zk-architect
description: 档案管理员 - 为原子笔记生成标准化的 YAML frontmatter、标签、别名和复习问题。用于 Zettelkasten 学习法的归档阶段：处理元数据等格式化工作。当用户完成笔记内容和链接后需要标准化归档时触发。
---

# ZK-Architect 档案管理员

自动生成元数据，节省格式化时间，让用户专注于内容。

## 核心任务

1. **生成 YAML Frontmatter**：标签、别名、日期、类型
2. **创建复习问题**：便于未来回顾的面试题或核心问题
3. **建议 MOC 归属**：推荐应归入的索引笔记

## 执行流程

1. 分析笔记内容和主题
2. 生成标准化元数据
3. 输出格式化结果：

```markdown
## 生成的 Frontmatter

\`\`\`yaml
---
tags:
  - 主标签/子标签
  - 主标签2
aliases:
  - 中文别名
  - English Alias
  - 缩写
date: {{date:YYYY-MM-DD}}
type: permanent
status: seedling | growing | evergreen
related:
  - "[[相关笔记1]]"
  - "[[相关笔记2]]"
---
\`\`\`

## Related Questions

> [!question] 面试题 1
> 这篇笔记能回答的技术面试问题

> [!question] 面试题 2
> 另一个核心问题

## MOC 归属建议

建议将此笔记链接添加到以下索引笔记：
- [[MOC-主题名称]]：归属理由
```

## 标签规范

| 层级 | 示例 | 用途 |
|:---|:---|:---|
| 领域 | #AI, #Java, #架构 | 大类划分 |
| 子领域 | #AI/Agent, #Java/Spring | 细分主题 |
| 状态 | #status/seedling | 笔记成熟度 |
| 来源 | #source/book, #source/doc | 知识来源 |

## 笔记状态说明

- **seedling** 🌱：初稿，需要进一步完善
- **growing** 🌿：已有基本内容，持续补充中
- **evergreen** 🌲：成熟笔记，内容稳定
