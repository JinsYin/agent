---
title: platform 只在镜像确无原生架构时声明
impact: MEDIUM
impactDescription: 多余的 platform 声明强制走模拟层，性能损失数倍
tags: compose, docker-compose, platform, arm64
---

## platform 只在镜像确无原生架构时声明

`platform: linux/amd64` 会强制该服务走模拟层（ARM 机器上的 QEMU）。数据库这类 IO 与 CPU 双密集的服务在模拟层下可能慢数倍，且偶发难以复现的兼容问题。

很多 `platform` 声明是历史遗留：当年该镜像确实只有 amd64，后来上游发布了多架构 manifest，声明却没人去掉。

**加之前先确认镜像是否真的没有原生变体：**

```bash
docker manifest inspect <image>:<tag> | grep architecture
```

有 `arm64` 就不要加。确需保留时，就地注释写明**为什么**——否则下一个人无从判断能不能删：

```yaml
  mysql:
    image: mysql:5.7
    platform: linux/amd64   # 5.7 官方镜像无 arm64 变体，Apple Silicon 必需
```

反例是给整个 compose 文件统一加 `platform`——那会把本可原生运行的服务一并拖进模拟层。
