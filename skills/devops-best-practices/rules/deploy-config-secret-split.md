---
title: 非敏感配置进 ConfigMap，敏感值进 Secret，用 envFrom 整体注入
impact: CRITICAL
impactDescription: 敏感值混进 ConfigMap 后会出现在 kubectl describe 与清单仓库里
tags: deploy, k8s, config, secret
---

## 非敏感配置进 ConfigMap，敏感值进 Secret，用 envFrom 整体注入

ConfigMap 的内容以明文出现在 `kubectl describe`、事件日志和清单仓库中。密码、密钥、token 一旦写进去，等于公开——`kubectl get configmap -o yaml` 是很多人默认有的权限。

判据不是"看起来重不重要"，而是**泄漏后是否需要轮换**：需要轮换的，就是 Secret。

**正确：**

```yaml
envFrom:
  - configMapRef:
      name: app-config      # 端点、超时、开关、日志级别
  - secretRef:
      name: app-secret      # 密码、密钥、token
```

用 `envFrom` 整体注入而非逐条 `env: - name/valueFrom`：新增一个配置项时只改 ConfigMap，不必同步改每个 Deployment——后者极易漏改其中一个服务，且漏改不报错，只是那个服务读到空值。

Secret 的清单文件本身**不入库**（见 `secret-never-commit-real`），仓库里只保留 `secret.example.yaml`。
