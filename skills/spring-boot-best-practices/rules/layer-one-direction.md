---
title: 分层单向依赖
impact: CRITICAL
tags: layer, architecture, controller, service, mapper
---

## 分层单向依赖

`Controller → Service → Mapper`，每层只依赖紧邻的下一层。反向或跨层调用会同时破坏事务边界、可测性与复用性。

| 层 | 职责 | 禁止 |
|---|---|---|
| Controller | HTTP 契约：路径、参数、校验、鉴权 | 访问 Mapper、调用其他 Controller |
| Service | 业务逻辑、事务、编排、外部调用、缓存 | 处理视图关注点、直接调别的 Service 的 Mapper |
| Mapper | 数据库访问、ORM 映射 | 业务逻辑、校验、事务 |

**错误（Controller 直连 Mapper，绕过事务与业务校验）：**

```java
@RestController
@RequiredArgsConstructor
public class UserController {
    private final UserMapper userMapper; // ❌ 跨层
    @GetMapping("/users/{id}")
    public R<UserEntity> get(@PathVariable Long id) {
        return R.ok(userMapper.selectById(id));
    }
}
```

**正确：**

```java
@RestController
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;
    @GetMapping("/users/{id}")
    public R<UserDetailResponse> get(@PathVariable Long id) {
        return R.ok(userService.getById(id));
    }
}
```
