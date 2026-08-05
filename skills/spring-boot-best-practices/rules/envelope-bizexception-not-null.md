---
title: 用 BizException 表达业务失败，不返回 null
impact: HIGH
tags: envelope, exception, error-handling
---

## 用 BizException 表达业务失败，不返回 null

用 `null` 表示「没找到」会把判空责任推给每个调用点，漏一处就是 NPE，而且丢失了失败原因。

**错误：**

```java
public UserDetailResponse getById(Long id) {
    UserEntity e = userMapper.selectById(id);
    return e == null ? null : converter.toResponse(e); // ❌
}
```

**正确：**

```java
public UserDetailResponse getById(Long id) {
    UserEntity e = userMapper.selectById(id);
    if (e == null) {
        throw new BizException(ErrorCode.USER_NOT_FOUND);
    }
    return converter.toResponse(e);
}
```

由 `GlobalExceptionHandler` 统一转成带码的 `R`。仅当「不存在」本身是正常分支（如可选配置查询）时才返回 `Optional`。
