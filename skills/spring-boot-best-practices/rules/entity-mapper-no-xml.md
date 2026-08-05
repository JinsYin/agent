---
title: Mapper 不写 XML 和整句 SQL
impact: HIGH
tags: entity, mapper, mybatis-plus, mpj, sql
---

## Mapper 不写 XML 和整句 SQL

Mapper 继承 `MPJBaseMapper<Entity>`，查询优先用 MyBatis-Plus wrapper 与 MPJ API（`selectJoinPage`、`leftJoin`、`selectAs`）。手写整句 SQL 会绑死方言，在多数据库目标下必然分叉。

**错误：**

```java
@Mapper
public interface UserMapper extends MPJBaseMapper<UserEntity> {
    @Select("SELECT * FROM t_user WHERE status = #{status} LIMIT 10") // ❌ 方言绑定
    List<UserEntity> listByStatus(String status);
}
```

**正确：**

```java
@Mapper
public interface UserMapper extends MPJBaseMapper<UserEntity> {
}

// Service 内用 wrapper
LambdaQueryWrapper<UserEntity> w = Wrappers.lambdaQuery(UserEntity.class)
        .eq(UserEntity::getStatus, status)
        .last("LIMIT 10");
```

复杂投影/聚合确实无法用 wrapper 表达时，允许在 wrapper 内嵌小段 SQL 片段，但必须参数化，并在旁边用注释写出等价 SQL 形状供审查者阅读。
