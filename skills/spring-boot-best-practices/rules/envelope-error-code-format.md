---
title: 错误码 6 位，前 3 位是 HTTP 状态
impact: HIGH
tags: envelope, error-code, api
---

## 错误码 6 位，前 3 位是 HTTP 状态

错误码固定 6 位，前 3 位复用对应的 HTTP 状态码，后 3 位是该状态下的序号。这样调用方不查表也能判断错误大类，网关和日志也能按前缀聚合。

**错误：**

```java
AUTH_INVALID(1001, "AK 或 SK 校验失败"), // ❌ 看不出属于哪类
```

**正确：**

```java
@Getter
@RequiredArgsConstructor
public enum ErrorCode {
    AUTH_INVALID(401001, 401, "AK 或 SK 校验失败"),
    AUTH_EXPIRED(401002, 401, "AK 已过期"),
    NOT_FOUND   (404001, 404, "资源不存在");

    private final int code;
    private final int httpStatus;
    private final String defaultMessage;
}
```

每个码自带默认中文 message，调用点不重复写文案。
