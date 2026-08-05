---
title: 跨领域调 Service，不调对方 Mapper
impact: HIGH
tags: layer, service, mapper, coupling
---

## 跨领域调 Service，不调对方 Mapper

一个 Service 可以注入另一个 Service，但**不能**注入另一个领域的 Mapper。绕过对方 Service 等于绕过它的业务校验、缓存失效和事务约定，这类 bug 只在对方逻辑变更时才暴露。

**错误：**

```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
    private final UserMapper userMapper;
    private final RoleMapper roleMapper; // ❌ 越过 RoleService
}
```

**正确：**

```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
    private final UserMapper userMapper;   // 自己领域的 Mapper
    private final RoleService roleService; // 别人领域走 Service
}
```
