---
title: 只对规范字节签名与摘要
impact: CRITICAL
impactDescription: 验签在对端随机失败
tags: crypto, sm2, sm3, signature, serialization
---

## 只对规范字节签名与摘要

签名和摘要的输入必须是**确定性字节序列**。对 `Map.toString()`、对象 `toString()` 或字段顺序不固定的 JSON 签名，会产生同一份数据在不同 JVM、不同版本下签出不同结果——表现为验签随机失败，且极难复现。

**错误：**

```java
byte[] data = payload.toString().getBytes(); // ❌ 顺序不保证
byte[] sig = signSm2Sm3(privateKey, data);
```

**正确（固定字段顺序的规范化序列化）：**

```java
byte[] canonical = canonicalize(payload); // 字段排序 + 固定编码 + UTF-8
byte[] sig = signSm2Sm3(privateKey, canonical);
```

对外来数据，**先验签再信任**。SM2 解密前先校验 `C1`、`C3`、`C2` 三段结构；`C1` 需含未压缩点前缀 `0x04`，除非对端协议明确不带。

摘要比较用常量时间比较，不要用 `Arrays.equals` 之外的短路比较暴露时序信息。
