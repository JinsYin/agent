---
title: 重复出现的仓库地址、tag 策略用 YAML anchor 收敛
impact: MEDIUM
impactDescription: 散落的副本改一处漏一处，且漏改的那个步骤照常成功
tags: ci, pipeline, yaml, dry
---

## 重复出现的仓库地址、tag 策略用 YAML anchor 收敛

同一个 registry 主机名在四个构建步骤里各写一遍，迁仓库时就要改四处。漏掉一处不会报错——那个步骤照常推送，只是推到了旧仓库，而清单从新仓库拉，表现为"镜像明明构建成功了但 Pod 拉不到"。

```yaml
global-variables:
  DOCKER_REGISTRY: &DOCKER_REGISTRY registry.internal
  REPO_ADMIN:      &REPO_ADMIN      registry.internal/org/app-admin
  # tag 策略：push 落 latest，打 tag 落版本号
  DOCKER_TAG:      &DOCKER_TAG      ${DRONE_TAG:-latest}

steps:
  - name: build-admin
    settings:
      registry: *DOCKER_REGISTRY
      repo: *REPO_ADMIN
      tags:
        - *DOCKER_TAG
```

收敛的判据是**是否会一起变**：registry 主机名、tag 策略、镜像源前缀会一起变，值得收敛；各服务的模块名不会一起变，不必收敛。

anchor 定义要集中在文件顶部并加注释说明用途。散落在各步骤中间定义的 anchor 比重复写更难读——读者得先找到定义处才知道 `*DOCKER_TAG` 是什么。
