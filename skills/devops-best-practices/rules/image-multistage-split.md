---
title: 构建与运行分阶段，运行阶段只带运行时
impact: CRITICAL
impactDescription: 单阶段镜像把编译器、源码与私库凭据一并带进产线
tags: image, dockerfile, multistage, security
---

## 构建与运行分阶段，运行阶段只带运行时

多阶段构建不只是为了减小体积，更是为了**缩小攻击面**。单阶段镜像会把 JDK/Node、构建工具、源码、以及构建时用到的私库凭据文件全部留在产线镜像层里——即便最后一层删掉了，前面的层仍然可被提取。

运行阶段选最小可用运行时：Java 用 JRE 而非 JDK，前端产物用 nginx 而非 Node。

**错误（构建产物与工具链混在一层，`.m2/settings.xml` 里的私库凭据永久留在镜像中）：**

```dockerfile
FROM maven:3.9.9-eclipse-temurin-21
COPY . .
RUN mvn package
ENTRYPOINT ["java", "-jar", "target/app.jar"]
```

**正确（builder 阶段的一切都不进入 runner）：**

```dockerfile
FROM ${BASE_REGISTRY}maven:3.9.9-eclipse-temurin-21 AS builder
WORKDIR /build
COPY . .
RUN mvn -pl ${MODULE} package -am

FROM ${BASE_REGISTRY}eclipse-temurin:21-jre AS runner
WORKDIR /app
COPY --from=builder /build/${MODULE}/target/${MODULE}.jar app.jar
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```
