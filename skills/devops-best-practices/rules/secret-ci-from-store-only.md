---
title: CI 里的凭据只能从 secret store 取，不写进 YAML
impact: CRITICAL
impactDescription: 流水线配置通常入库且日志公开，明文凭据等同公开
tags: secret, ci, credentials
---

## CI 里的凭据只能从 secret store 取，不写进 YAML

CI 配置文件（`.drone.yml` / `.gitlab-ci.yml` / workflow）是入库的，而且构建日志往往对整个团队可见。凭据写进去就是双重泄漏。

各家的取法不同但形态一致——声明引用，不写值：

```yaml
# Drone
username:
  from_secret: docker_username

# GitHub Actions
password: ${{ secrets.REGISTRY_PASSWORD }}

# GitLab CI —— 在项目 CI/CD Variables 中定义，YAML 里只用变量名
```

**注意插件与原生命令的差异**：许多 CI 插件支持 `settings.<key>.from_secret`，但你自己写的 `commands:` 步骤不走插件的 secret 解析。这时必须先经 `environment` 注入成环境变量，再在命令里引用：

```yaml
- name: deploy-sdk
  image: maven:3.9.9-eclipse-temurin-21
  environment:
    NEXUS_PASSWORD:
      from_secret: nexus_password
  commands:
    - mvn deploy -Dnexus.password=$NEXUS_PASSWORD
```

命令里引用环境变量而非直接内插密文，可避免它出现在 `set -x` 的回显中。
