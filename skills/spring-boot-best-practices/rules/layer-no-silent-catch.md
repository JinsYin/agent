---
title: 禁止静默吞异常
impact: HIGH
impactDescription: 故障无声传播
tags: layer, exception, error-handling
---

## 禁止静默吞异常

`catch` 后不处理也不重抛，会把失败伪装成成功：调用方收到 200 和空数据，故障延后到下游才暴露，且现场已丢失。

**错误：**

```java
try {
    userMapper.insert(entity);
} catch (Exception e) {
    // ❌ 什么都不做，调用方以为成功
}
```

**正确（要么转成有业务含义的错误，要么交给全局处理器）：**

```java
try {
    externalClient.sync(entity);
} catch (RestClientException e) {
    log.warn("上游同步失败, entityId={}", entity.getId(), e);
    throw new BizException(ErrorCode.UPSTREAM_UNAVAILABLE);
}
```

只有明确「失败可忽略」的旁路逻辑（如埋点上报）才允许吞，且必须留日志并注释原因。
