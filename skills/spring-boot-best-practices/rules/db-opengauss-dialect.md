---
title: openGauss 方言三定律
impact: HIGH
tags: db, migration, opengauss, gaussdb, sql
---

## openGauss 方言三定律

写 gauss 系迁移时，以下 PostgreSQL 语法**不可用**，必须改写：

| 不支持 | 改写为 |
|---|---|
| `ON CONFLICT` | `INSERT ... WHERE NOT EXISTS`，或显式去重 |
| `sys_guid()` / `gen_random_uuid()` | `md5(...)::uuid` 合成 |
| `jsonb_build_object` | `'[...]'::jsonb` 字面量 |

**错误：**

```sql
INSERT INTO t_config(k, v) VALUES ('a', '1')
ON CONFLICT (k) DO UPDATE SET v = '1'; -- ❌ openGauss 不支持
```

**正确：**

```sql
INSERT INTO t_config(k, v)
SELECT 'a', '1'
WHERE NOT EXISTS (SELECT 1 FROM t_config WHERE k = 'a');
```

这三条是踩坑实证，不是理论限制——写之前先对照，比迁移失败后回查便宜得多。
