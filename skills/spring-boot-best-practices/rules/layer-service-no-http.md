---
title: Service 不碰 HTTP
impact: HIGH
tags: layer, service, http, testability
---

## Service 不碰 HTTP

Service 里出现 `HttpServletRequest`、`request.getParameter()`、手工构造响应头或状态码，会让业务逻辑绑死在 Web 容器上——单测必须起 MockMvc，定时任务和 MQ 消费者也没法复用同一段逻辑。

**错误：**

```java
public boolean create(HttpServletRequest request) { // ❌
    String name = request.getParameter("name");
}
```

**正确（Controller 完成解析与鉴权，Service 只收类型化入参）：**

```java
public boolean create(UserCreateRequest request, Long operatorId) {
    // 纯业务逻辑，可被 Controller / 定时任务 / MQ 消费者共用
}
```

当前登录人这类上下文由 Controller 取出后作为参数传入，或经由显式的上下文持有者，不要让 Service 直接读 HTTP。
