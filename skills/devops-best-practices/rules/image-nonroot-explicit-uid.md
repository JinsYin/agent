---
title: 非 root 运行，且 UID/GID 必须显式指定
impact: CRITICAL
impactDescription: 隐式分配的系统 UID 与 K8s securityContext 不匹配，挂载卷写不进去
tags: image, dockerfile, security, uid
---

## 非 root 运行，且 UID/GID 必须显式指定

容器以 root 运行会在逃逸时直接拿到宿主权限。但仅仅"创建一个用户"不够——**UID 数值必须显式指定并与部署清单一致**。

`useradd -r` 分配的是系统 UID（通常 100–999），而 K8s 的 `runAsUser: 1000` / `fsGroup: 1000` 期待 1000。两者不匹配时，`fsGroup` 对挂载卷执行 chown 后，进程反而失去写权限——报错是"权限拒绝"，根因却在 Dockerfile 里，排查代价很高。

**基础镜像陷阱**：Ubuntu 24.04（noble）系基础镜像（含 `eclipse-temurin:21-jre`）已预置 `ubuntu` 用户占用 UID/GID 1000，直接 `groupadd -g 1000` 会因 GID 被占而失败（exit 4）。必须先释放。

**错误（UID 由系统分配，值不可控）：**

```dockerfile
RUN useradd -r -s /usr/sbin/nologin app
USER app
```

**正确（先释放占位用户，再钉死 1000）：**

```dockerfile
# userdel 对非 Ubuntu 基础镜像是无害 no-op
RUN userdel -r ubuntu 2>/dev/null || true \
 && groupadd -g 1000 app \
 && useradd -u 1000 -g 1000 -m -s /usr/sbin/nologin app

COPY --chown=1000:1000 --from=builder /build/target/app.jar app.jar
USER app
```

部署清单侧必须同步（见 `deploy-securitycontext-match-image`）。
