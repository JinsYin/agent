---
title: Controller 不得返回 Entity
impact: CRITICAL
impactDescription: 敏感字段泄漏
tags: layer, controller, entity, response, security
---

## Controller 不得返回 Entity

Entity 映射数据库全字段，含密码散列、软删标记、内部审计列等。直接返回等于把这些字段暴露给调用方，且今后给表加一列就会静默扩大接口输出。

**错误（把整行数据抛给前端）：**

```java
@GetMapping("/users/{id}")
public R<UserEntity> get(@PathVariable Long id) {
    return R.ok(userService.getEntityById(id)); // ❌ 含 password / deleted
}
```

**正确（用视图专属 Response，字段显式列举）：**

```java
@GetMapping("/users/{id}")
public R<UserDetailResponse> get(@PathVariable Long id) {
    return R.ok(userService.getById(id));
}
```

Service 的返回类型也必须是 `Response` / `Dto` / 原始类型，**不能**是 Entity——否则 Controller 只是把泄漏点上移一层。
