---
title: base 放公共资源，overlay 只放环境差量
impact: HIGH
impactDescription: overlay 里复制整份清单后，base 的修复不会传播到各环境
tags: deploy, k8s, kustomize, helm
---

## base 放公共资源，overlay 只放环境差量

Kustomize overlay（或 Helm 的 values 文件）的价值全在于**差量**。一旦某个 overlay 复制了整份 Deployment 而不是 patch，它就与 base 脱钩：后续在 base 上修的探针、安全上下文、标签，都不会传播到这个环境。

而这种脱钩不报错——`kubectl apply` 照样成功，只是这个环境悄悄停在了旧版本。

overlay 里应当只出现这几类：命名空间、副本数、资源限额、镜像 tag、域名/IP、环境标签。**其余一律回 base。**

**正确：**

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base
  - secret.yaml

namespace: app-prod
commonLabels:
  env: prod

images:
  - name: registry.internal/org/app
    newTag: 1.4.2

patches:
  - path: replicas-patch.yaml
```

若发现某个 overlay 的 patch 越来越大，说明该差异其实应该参数化进 base，而不是继续堆 patch。
