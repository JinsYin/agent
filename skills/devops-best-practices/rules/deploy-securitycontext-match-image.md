---
title: securityContext 的 UID 必须与镜像内创建的用户一致
impact: CRITICAL
impactDescription: 两处 UID 不一致时挂载卷 chown 后进程失去写权限
tags: deploy, k8s, security, uid
---

## securityContext 的 UID 必须与镜像内创建的用户一致

`runAsUser` 不会去镜像里查用户是否存在——它直接以该数值运行进程。若镜像里创建的是 UID 1001 而清单写 1000，进程会以一个镜像内**不存在的用户**运行：`whoami` 报错，`$HOME` 不存在，任何按用户名解析权限的逻辑都会失败。

挂载卷时更隐蔽：`fsGroup` 会把卷 chown 成指定 GID，若与进程实际 GID 不符，进程反而写不进自己的数据目录。

**正确（两处同源，并在注释里点明对应关系）：**

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000     # 与 Dockerfile 中 useradd -u 1000 一致
    runAsGroup: 1000
    # 无 PVC 时不要加 fsGroup——它会对所有卷做递归 chown，大卷上开销显著
```

`runAsNonRoot: true` 是必备的兜底：镜像若忘了 `USER` 指令，Pod 会直接拒绝启动，而不是悄悄以 root 跑起来。
