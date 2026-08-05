---
title: 三探针分工明确，慢启动用 startupProbe 而非调大 initialDelay
impact: CRITICAL
impactDescription: 用 initialDelaySeconds 兜慢启动，会让故障期同样延迟被发现
tags: deploy, k8s, probe, health
---

## 三探针分工明确，慢启动用 startupProbe 而非调大 initialDelay

三个探针职责不同，不能互相替代：

| 探针 | 失败后果 | 管什么 |
|---|---|---|
| `startupProbe` | 重启容器 | 启动窗口，通过前另两个探针**不生效** |
| `livenessProbe` | 重启容器 | 进程死锁/僵死 |
| `readinessProbe` | 摘出 Service 端点 | 暂时不能接流量（依赖未就绪、正在预热） |

JVM、大型 Node 应用启动慢，常见的错误处置是把 `livenessProbe.initialDelaySeconds` 调到 120s 以上。副作用是**运行期的故障同样要等 120s 才被发现**——用启动期的宽容度买单了运行期的敏感度。

正确解法是用 `startupProbe` 单独覆盖启动窗口：它通过之前 liveness/readiness 都不生效，通过之后 liveness 立刻以正常的短周期工作。

**错误（启动宽容度污染了运行期）：**

```yaml
livenessProbe:
  httpGet: { path: /actuator/health/liveness, port: 8080 }
  initialDelaySeconds: 180
  periodSeconds: 15
```

**正确（启动窗口 = failureThreshold × periodSeconds = 300s）：**

```yaml
startupProbe:
  httpGet: { path: /actuator/health/liveness, port: 8080 }
  initialDelaySeconds: 10
  periodSeconds: 10
  failureThreshold: 30

livenessProbe:
  httpGet: { path: /actuator/health/liveness, port: 8080 }
  periodSeconds: 15
  failureThreshold: 3

readinessProbe:
  httpGet: { path: /actuator/health/readiness, port: 8080 }
  periodSeconds: 10
  failureThreshold: 3
```

liveness 与 readiness 必须指向**不同**的端点：liveness 只答"进程还活着吗"，readiness 才检查下游依赖。两者指向同一个含依赖检查的端点时，下游抖动会导致 Pod 被反复重启，而重启并不能修复下游。
