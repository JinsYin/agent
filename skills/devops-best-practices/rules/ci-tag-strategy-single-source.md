---
title: 镜像 tag 策略单一来源，部署清单与 CI 对齐
impact: HIGH
impactDescription: 两侧各自维护 tag 时，部署拉到的不是刚构建的那个镜像
tags: ci, deploy, image-tag, release
---

## 镜像 tag 策略单一来源，部署清单与 CI 对齐

CI 决定推什么 tag，部署清单决定拉什么 tag。这两个决定必须来自同一条规则，否则就会出现"构建成功、部署成功、但跑的是上一个版本"——最难查的一类问题，因为每一步都显示绿色。

先把策略写成一句话，两侧都遵守它：

> push 到主干 → `latest`（预发环境消费）；打 tag → 版本号（生产消费）。

```yaml
# CI 侧
DOCKER_TAG: &DOCKER_TAG ${DRONE_TAG:-latest}
```

```yaml
# 部署侧（预发 overlay）
images:
  - name: registry.internal/org/app
    newTag: latest            # 对齐 CI：push 落 latest

# 生产 overlay
images:
  - name: registry.internal/org/app
    newTag: 1.4.2             # 对齐 CI：tag 落版本号
```

**生产侧的版本号应由发版脚本统一 bump**，不要人工在多个文件里各改一遍——服务数量一多，漏改一个的概率接近必然。若同时维护 Kustomize 与 Helm 两套清单，两者的版本必须一起改，并在文档里点明这个联动关系。
