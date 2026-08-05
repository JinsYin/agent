---
title: R / RList / RPage 按形状选型
impact: HIGH
tags: envelope, response, pagination
---

## R / RList / RPage 按形状选型

| 返回形状 | 类型 |
|---|---|
| 单对象 | `R<{Name}Response>` |
| 不分页列表 | `RList<{Name}Response>` |
| 分页列表 | `RPage<{Name}Response>` |
| 仅成功/失败 | `R<Boolean>` |
| 成功无数据 | `R<Void>` + `R.ok()` |

选错类型会让前端拿不到 `total` / `pageNo`，或被迫解包多余的一层。

**错误：**

```java
public R<List<UserResponse>> page(UserPageQuery q) { // ❌ 分页信息丢失
    return R.ok(userService.page(q).getData());
}
```

**正确：**

```java
public RPage<UserResponse> page(@Valid UserPageQuery q) {
    return RPage.ok(userService.page(q));
}
```

Service 层分页返回 `PageResult<T>`，由 Controller 转成 `RPage<T>`。
