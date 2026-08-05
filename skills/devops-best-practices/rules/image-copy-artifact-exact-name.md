---
title: COPY 构建产物用确定文件名，不靠通配猜
impact: MEDIUM
impactDescription: 通配匹配不到时构建在最后一步失败，前面的耗时全部浪费
tags: image, dockerfile, copy, artifact
---

## COPY 构建产物用确定文件名，不靠通配猜

`COPY --from=builder /build/target/app-*.jar app.jar` 依赖构建产物的命名恰好带版本号后缀。而构建配置（Maven 的 `<finalName>`、Vite 的 `build.outDir`）随时可能把它改掉——改完通配就匹配不到，`COPY` 失败。

代价在于**失败时机**：它发生在完整编译之后的最后一层，前面几分钟的构建全部作废，而错误信息只说找不到文件，不会提示是命名规则变了。

**错误（假定产物名含版本号）：**

```dockerfile
COPY --from=builder /build/target/app-*.jar app.jar
```

**正确（与构建配置约定确定名，用 ARG 参数化模块）：**

```dockerfile
ARG MODULE=app
# pom 中 <finalName>${project.artifactId}</finalName> → 产物即 ${MODULE}.jar
COPY --chown=1000:1000 --from=builder /build/${MODULE}/target/${MODULE}.jar app.jar
```

若确实无法固定命名，就在 builder 阶段先重命名成确定名，再在 runner 阶段按确定名 COPY。
