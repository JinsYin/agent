---
title: 易错的启动前提写成守护进程，而不是写进文档
impact: MEDIUM
impactDescription: 靠文档约束的前提必然被遗忘，且违反后的症状远离根因
tags: proc, process-compose, guard, dx
---

## 易错的启动前提写成守护进程，而不是写进文档

有些启动前提违反后不会立刻失败，而是在很远的地方以另一种形态爆出来：编译时漏了某个 profile，直到运行期连数据库才报协议不兼容；JDK 版本不对，报错停在某个无关的编译错误上。

这类前提写进 README 是无效的——需要它的人正好是不会去读的人。应当写成一个**启动链最前端的守护进程**，让违反直接变成一句清楚的报错。

```yaml
processes:
  profile-guard:
    command: "bash ./scripts/local-startup-guard.sh"
    availability:
      restart: "no"

  build-common:
    depends_on:
      profile-guard:
        condition: process_completed_successfully
```

守护脚本要做到两点：**退出码明确**（0 通过 / 非 0 阻断），**报错信息直接给出修法**而不只是说哪里不对。

守护的对象应是"自动化能查、人容易忘"的前提：必需的构建参数、工具版本、依赖服务是否在跑、必需的环境变量是否已设。不要守护那些本来就会立刻失败的东西——那是重复劳动。
