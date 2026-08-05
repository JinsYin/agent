---
title: 构建在指定镜像内进行，不依赖 CI 节点自带环境
impact: HIGH
impactDescription: 依赖节点环境时，换节点或节点升级会让构建结果无声改变
tags: ci, pipeline, reproducibility
---

## 构建在指定镜像内进行，不依赖 CI 节点自带环境

若构建步骤直接调用节点上的 `mvn` / `node`，那么"用哪个版本编译"就取决于节点当时装了什么。加节点、升级节点、或换一台跑，产出就可能不同——而这类差异不报错，只表现为"在某台机器上构建出来的包有问题"。

每个构建步骤都要显式指定镜像，且**版本号钉到次版本以上**：

```yaml
- name: deploy-sdk
  image: maven:3.9.9-eclipse-temurin-21     # 不写 maven:latest
  commands:
    - mvn deploy
```

同一条流水线里跨步骤的工具版本必须一致。后端用 temurin-21 构建、镜像里却是 temurin-17 运行，属于典型的跨步骤漂移。

**基础镜像的版本也要与本地对齐**：CI 用 Node 24、本地用 Node 20，`pnpm install --frozen-lockfile` 可能因原生模块的预编译产物不同而结果不同。锁定版本的价值就在这里。
