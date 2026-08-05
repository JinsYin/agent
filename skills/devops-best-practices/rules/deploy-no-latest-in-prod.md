---
title: 生产环境不用 latest，镜像 tag 必须确定
impact: HIGH
impactDescription: latest 使"当前跑的是哪个版本"不可知，回滚也无从指定目标
tags: deploy, k8s, image-tag, release
---

## 生产环境不用 latest，镜像 tag 必须确定

`latest` 让部署结果依赖"拉取那一刻仓库里是什么"。同一份清单在两个时间点 apply 会得到不同的镜像，而清单本身没有任何差异——事故复盘时无法回答"当时跑的是哪个提交"。

回滚更直接受害：`kubectl rollout undo` 回退的是清单版本，若前后清单都写着 `latest`，回退后拉到的仍是同一个镜像。

| 环境 | tag | 理由 |
|---|---|---|
| 本地 / 开发 | `latest` 可接受 | 追最新，且随时可重建 |
| 测试 / 预发 | `latest` 可接受 | 持续集成的目标就是最新 |
| **生产** | **必须确定版本号** | 可追溯、可回滚 |

生产用确定 tag 时，`imagePullPolicy` 应为 `IfNotPresent`（tag 不可变，无需每次拉取）；用 `latest` 的环境则必须 `Always`，否则节点上的旧缓存会让"最新"名不副实。

tag 的取值应与 CI 的 tag 策略同源（见 `ci-tag-strategy-single-source`），不要人工在清单里另起一套。
