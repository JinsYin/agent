---
title: exec 形态的 ENTRYPOINT 不展开环境变量
impact: HIGH
impactDescription: 注入的 JAVA_OPTS 等变量静默失效，不报错
tags: image, dockerfile, entrypoint, env
---

## exec 形态的 ENTRYPOINT 不展开环境变量

`ENTRYPOINT ["java", "-jar", "app.jar"]` 是 exec 形态，**不经过 shell**，因此 `$JAVA_OPTS` 这类变量不会被展开——它会被当作字面量参数传进去，或者干脆不出现。

这是个沉默故障：容器正常启动、日志正常，只是你注入的 JVM 参数（时区、内存、GC）一个都没生效。

exec 形态仍是推荐默认（进程为 PID 1，能正确接收 SIGTERM）。所以正确解法**不是**改成 shell 形态，而是在需要注入变量的地方显式套一层 shell。

**错误（ConfigMap 注入的 JAVA_OPTS 完全不生效）：**

```dockerfile
ENTRYPOINT ["java", "$JAVA_OPTS", "-jar", "/app/app.jar"]
```

**正确（镜像保持 exec 形态；部署清单需要注入时覆盖 command/args）：**

```dockerfile
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

```yaml
# K8s 侧：exec 套 shell，用 exec 保证 java 仍是 PID 1
command: ["/bin/sh", "-c"]
args: ["exec java $JAVA_OPTS -jar /app/app.jar"]
```
