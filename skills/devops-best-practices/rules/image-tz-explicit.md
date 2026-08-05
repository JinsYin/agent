---
title: 显式声明时区，alpine 需额外装 tzdata
impact: MEDIUM
impactDescription: 容器默认 UTC，日志与业务时间与预期相差整数小时
tags: image, dockerfile, timezone
---

## 显式声明时区，alpine 需额外装 tzdata

容器默认时区是 UTC，与宿主机无关。不显式声明，日志时间戳、定时任务触发点、以及任何依赖"当天"边界的业务逻辑都会偏移。

**glibc 系镜像**（debian/ubuntu 基础，含 temurin）通常已内置 tzdata，设 `ENV TZ` 即可。**musl 系镜像**（alpine）默认不含 tzdata，只设 `TZ` 无效——必须显式安装。

```dockerfile
# glibc 系：设置即可
ENV TZ=Asia/Shanghai

# musl 系（alpine）：必须补装 tzdata
ENV TZ=Asia/Shanghai
RUN apk add --no-cache tzdata
```

**JVM 应用要额外注意**：JVM 读 `TZ` 决定默认时区，但一旦通过 `JAVA_OPTS` 传了 `-Duser.timezone`，后者优先。两处都设时必须一致，否则容器时区与 JVM 时区不同——日志（容器时区）和业务时间（JVM 时区）会对不上，这种不一致极难排查。
