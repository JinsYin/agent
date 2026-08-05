---
title: 可选与备用服务挂 profiles，默认不启动
impact: HIGH
impactDescription: 备用服务默认起来后与主服务抢端口、抢资源，且新人以为它是必需的
tags: compose, docker-compose, profile
---

## 可选与备用服务挂 profiles，默认不启动

`docker compose up -d` 应该恰好起**跑起项目所必需的那些**服务。备用数据库、可选中间件、调试工具若默认启动，会抢端口、吃内存，还会让新人误以为它们是架构的一部分。

```yaml
services:
  opengauss:            # 默认库，无 profiles → 零参数即启动
    image: opengauss/opengauss:5.0.0

  mysql:                # 备用库，仅在显式指定时启动
    profiles: [mysql]
    image: mysql:5.7
```

```bash
docker compose up -d                    # 只起默认集
docker compose --profile mysql up -d    # 显式追加备用库
```

**在文件头部用注释写明默认起哪些、备用怎么起。** compose 文件读者第一时间想知道的就是这个，而 `profiles` 分散在各服务定义里，通读一遍才能拼出全貌。

同时确保项目文档里的启动命令与此一致——文档说"起 MySQL"而实际默认起的是别的库，是最常见的一类文档漂移。
