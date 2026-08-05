---
title: 信封只用工厂方法构造
impact: HIGH
tags: envelope, response, api
---

## 信封只用工厂方法构造

`R` / `RList` / `RPage` 一律用静态工厂构造，不要 `new` 后逐字段赋值——手工赋值必然出现 code 取值不一致、message 漏填。

**错误：**

```java
R<UserResponse> r = new R<>(); // ❌
r.setCode(0);                  // 与工厂的 200 不一致
r.setData(user);
return r;
```

**正确：**

```java
return R.ok(user);                        // 成功带数据
return R.ok();                            // 成功无数据
return R.error(ErrorCode.DUPLICATE_NAME); // 带码错误
throw new BizException(ErrorCode.NOT_FOUND); // 交给全局处理器包装
```

Controller 直接返回 `R<T>`，不要套 `ResponseEntity<R<T>>`。仅文件下载与重定向需要自定 HTTP 状态时才用 `ResponseEntity<Resource>` 或 `void` + `HttpServletResponse`。
