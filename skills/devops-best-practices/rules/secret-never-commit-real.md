---
title: 真凭据永不入库，仓里只留 example 模板
impact: CRITICAL
impactDescription: 提交后即使删除，git 历史里仍永久可取
tags: secret, git, credentials, security
---

## 真凭据永不入库，仓里只留 example 模板

提交过的凭据不会因为下一个提交删掉它而消失——`git log -p`、任何一份克隆、以及所有 fork 里都还在。**唯一正确的处置是当作已泄漏，立刻轮换**，而不是删掉了事。

因此约束必须前移到"不可能提交"，而不是"记得别提交"：

```gitignore
# 真凭据文件，永不入库
deploy/k8s/overlays/*/secret.yaml
deploy/release/systemd/*.env
.env

# 模板必须入库（否则新人不知道要配哪些项）
!deploy/k8s/base/secret.example.yaml
!.env.example
```

模板与真文件成对存在：模板入库、列全所有键、值全部为占位；真文件被忽略、由部署者本地填。

**新增一个敏感配置项时，两个文件都要改。** 只改真文件不改模板，下一个部署的人会缺这一项——而缺失的表现通常是运行期某个功能静默不工作，不是启动失败。
