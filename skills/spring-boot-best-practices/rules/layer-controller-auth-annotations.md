---
title: 权限校验用注解，不写在方法体里
impact: HIGH
impactDescription: 漏判等于接口裸奔
tags: layer, controller, auth, sa-token, security
---

## 权限校验用注解，不写在方法体里

鉴权用 Sa-Token 注解声明，不在方法体里手写 if 判断。声明式的好处是**默认拒绝**：类上标了 `@SaCheckLogin`，新加的方法自动继承；手写判断则是默认放行，新方法忘了写就是裸奔，而且这种遗漏在测试里通常发现不了。

| 注解 | 用途 |
|---|---|
| `@SaCheckLogin` | 要求已登录，通常标在类上 |
| `@SaCheckPermission("user:query")` | 要求具体权限点 |
| `@SaCheckRole("admin")` | 要求角色 |
| `@SaIgnore` | 显式放行公开接口 |

公开接口必须用 `@SaIgnore` **显式**标注，而不是靠「类上没加校验」隐式放行——显式标注让审查者一眼看出这是有意为之。

**正确：**

```java
@Tag(name = "用户")
@SaCheckLogin
@RestController
@RequestMapping("/users")
@Validated
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @Operation(summary = "分页查询用户")
    @GetMapping
    @SaCheckPermission("user:query")
    public RPage<UserListItemResponse> page(@Valid UserPageQuery query) {
        return RPage.ok(userService.listUsers(query));
    }
}
```
