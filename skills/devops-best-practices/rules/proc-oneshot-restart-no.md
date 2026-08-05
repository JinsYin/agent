---
title: 一次性进程必须声明 restart:"no" 并被 depends_on 条件消费
impact: HIGH
impactDescription: 默认重启策略会让构建类进程无限循环执行
tags: proc, process-compose, lifecycle
---

## 一次性进程必须声明 restart:"no" 并被 depends_on 条件消费

process-compose 里既有长驻服务，也有"跑完就该结束"的前置进程（编译公共模块、跑迁移、环境自检）。后者若沿用默认重启策略，成功退出会被判为异常终止而重新拉起——形成无限构建循环，CPU 打满而看起来"还在启动中"。

```yaml
processes:
  build-common:
    command: "./mvnw -pl common install"
    availability:
      restart: "no"        # 一次性进程，成功退出后不再拉起

  app:
    command: "./mvnw -pl app spring-boot:run"
    depends_on:
      build-common:
        condition: process_completed_successfully
    availability:
      restart: on_failure  # 长驻服务，崩溃时重拉
```

`condition` 要选对：`process_completed_successfully` 才会等它**成功**结束；`process_started` 只等它开始，前置构建还没跑完服务就起来了，等于没配依赖。

长驻服务用 `on_failure` 而非 `always`——`always` 会在你主动停掉某个进程时又把它拉起来，调试时很碍事。
