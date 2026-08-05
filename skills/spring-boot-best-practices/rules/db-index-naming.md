---
title: 索引命名 uk_ / idx_
impact: MEDIUM
tags: db, naming, index
---

## 索引命名 uk_ / idx_

- 唯一索引：`uk_{table}_{field...}`
- 普通索引：`idx_{table}_{field...}`

带表名是为了让索引名全库唯一——某些数据库的索引名是 schema 级而非表级，重名会在建表时直接冲突。

```sql
CREATE UNIQUE INDEX uk_user_email ON t_user(email);
CREATE INDEX idx_order_created_at ON t_order(created_at);
```

新建表的唯一约束写成 `CREATE TABLE` 内联 `CONSTRAINT uk_x UNIQUE(...)`，而不是建表后再补索引——内联写法在分布式与单机部署下语义一致。
