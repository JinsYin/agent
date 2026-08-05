---
title: 审计时间字段用 @TableField(fill)
impact: CRITICAL
impactDescription: 跨方言下时间字段为 NULL
tags: entity, mybatis-plus, audit, opengauss
---

## 审计时间字段用 @TableField(fill)

不能依赖数据库的 `ON UPDATE CURRENT_TIMESTAMP`——该语法是 MySQL 特有的，迁移到 gauss 系时会被移除，字段随即变成 NULL 或永不更新。填充必须由应用层的 `MetaObjectHandler` 驱动。

**错误（指望数据库自动维护）：**

```java
private LocalDateTime createdAt; // ❌ 无 fill，靠 DDL 默认值
private LocalDateTime updatedAt;
```

**正确：**

```java
@TableField(fill = FieldFill.INSERT)
private LocalDateTime createdAt;

@TableField(fill = FieldFill.INSERT_UPDATE)
private LocalDateTime updatedAt;
```

规则固定：含 `createdAt` 必标 `INSERT`，含 `updatedAt` 必标 `INSERT_UPDATE`。
