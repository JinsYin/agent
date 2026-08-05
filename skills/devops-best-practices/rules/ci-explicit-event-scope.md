---
title: 每个步骤显式限定触发事件，不靠继承流水线级条件
impact: HIGH
impactDescription: 新增事件类型时，未限定的步骤会被意外触发
tags: ci, pipeline, trigger, event
---

## 每个步骤显式限定触发事件，不靠继承流水线级条件

流水线级的 `trigger` 决定"这次要不要跑"，步骤级的 `when` 决定"这一步该不该参与这次跑"。只写前者时，所有步骤对所有事件一视同仁。

问题出在**新增事件类型的那一刻**：为了加一个只需执行轻量操作的事件（如 promote 一个部署标记、或手动触发一次数据同步），流水线级 `trigger` 被扩了一项，结果所有构建步骤跟着跑了一遍——重建镜像、重新推送、覆盖 tag。这不是假想，是每次扩 trigger 都会遇到的默认行为。

**正确（构建步骤只认 push/tag，轻量步骤只认它自己的事件）：**

```yaml
trigger:
  event: [push, tag, promote]

steps:
  - name: build-image
    when:
      event: [push, tag]          # promote 时不重建

  - name: write-deploy-marker
    when:
      event: [promote]
      target: [sync-db-pre]       # 进一步限定 promote 的目标
```

**新增任何步骤时，第一件事就是写 `when`。** 遗漏不会报错，只会在下一次非常规事件时以"为什么它又重新构建了"的形式暴露。
