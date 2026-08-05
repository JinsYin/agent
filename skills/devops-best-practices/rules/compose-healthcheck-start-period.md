---
title: 依赖服务必须配 healthcheck，且 start_period 覆盖真实启动耗时
impact: HIGH
impactDescription: 没有健康检查时应用会连上尚未就绪的数据库，报错指向应用而非依赖
tags: compose, docker-compose, healthcheck
---

## 依赖服务必须配 healthcheck，且 start_period 覆盖真实启动耗时

容器"已启动"不等于"可服务"。数据库进程起来到能接受连接之间可能有几十秒，期间应用连上去会拿到连接拒绝或认证失败——**报错指向应用，根因在依赖**，这是本地环境最常见的误诊。

`start_period` 是关键参数：它期间的失败**不计入** `retries`，专门用于覆盖初始化窗口。首次初始化（建库、建表、导入种子数据）往往比后续启动慢一个数量级，`start_period` 要按首次算。

```yaml
healthcheck:
  # 探真实可服务性，不要只探端口
  test: ["CMD-SHELL", "gsql -U dap -d dap -c 'SELECT 1;' -h 127.0.0.1 || exit 1"]
  interval: 10s
  timeout: 10s
  retries: 15
  start_period: 90s     # 首次初始化建库建表，按最慢路径给
```

**探针要探真实可服务性**：`nc -z localhost 5432` 只证明端口在监听，而数据库在恢复期同样监听端口却拒绝查询。执行一条真实查询才是有效探测。

依赖方用 `depends_on` 的 `condition: service_healthy` 消费这个结果，否则 healthcheck 配了也只是好看。
