---
title: 主键用 ASSIGN_ID，禁用 AUTO
impact: CRITICAL
impactDescription: INSERT 直接失败
tags: entity, mybatis-plus, primary-key, opengauss
---

## 主键用 ASSIGN_ID，禁用 AUTO

`IdType.AUTO` 依赖数据库端的自增列。当目标库不是 MySQL 时（openGauss 的 `BIGSERIAL` 语义与 `AUTO_INCREMENT` 并不等价），应用层不填 ID，INSERT 撞上 NOT NULL 直接失败——而报错信息指向字段约束，与真正的根因（主键策略）相距很远，排查代价很高。

用 `ASSIGN_ID` 由应用层雪花算法生成，与数据库无关。

**错误：**

```java
@TableId(type = IdType.AUTO) // ❌ 绑死数据库自增
private Long id;
```

**正确：**

```java
@TableId(type = IdType.ASSIGN_ID)
private Long id;
```

**例外**——主键直接复用业务 ID 时用 `INPUT`：

```java
@TableId(value = "app_id", type = IdType.INPUT)
private Long appId;
```
