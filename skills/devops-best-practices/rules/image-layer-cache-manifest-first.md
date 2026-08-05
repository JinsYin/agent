---
title: 先复制依赖清单再装依赖，最后复制源码
impact: HIGH
impactDescription: 顺序颠倒导致改一行源码就重装全部依赖
tags: image, dockerfile, cache, build-speed
---

## 先复制依赖清单再装依赖，最后复制源码

Docker 按层缓存，任一层的输入变了，**该层及其后所有层全部失效**。源码的变更频率远高于依赖清单，所以必须把 `COPY 源码` 放在 `install 依赖` 之后。

顺序写反不会报错，只是每次构建都重装依赖——本地感觉"有点慢"，在 CI 上是每次推送多几分钟。

**错误（改任意一行源码都会重新 install）：**

```dockerfile
COPY . .
RUN pnpm install --frozen-lockfile
RUN pnpm build
```

**正确（依赖层只在清单变化时失效）：**

```dockerfile
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

COPY . ./
RUN pnpm build
```

依赖安装必须用 lockfile 的严格模式（`--frozen-lockfile` / `npm ci` / `mvn -o`），否则缓存命中了也可能装出与本地不同的版本。
