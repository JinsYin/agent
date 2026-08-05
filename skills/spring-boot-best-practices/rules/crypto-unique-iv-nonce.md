---
title: IV / nonce 每次随机且唯一
impact: CRITICAL
impactDescription: 固定 nonce 使 GCM 完全失效
tags: crypto, sm4, gcm, iv, nonce
---

## IV / nonce 每次随机且唯一

- SM4 密钥必须恰好 128 位 / 16 字节
- GCM nonce 96 位 / 12 字节，**每次加密唯一**
- CBC IV 16 字节，每次用 `SecureRandom` 生成

GCM 下重用 nonce 不只是削弱强度——它让攻击者能恢复认证密钥，从而伪造任意密文的标签，加密保护完全失效。

**错误：**

```java
private static final byte[] IV = "1234567890123456".getBytes(); // ❌ 固定
```

**正确：**

```java
byte[] nonce = new byte[12];
SecureRandom.getInstanceStrong().nextBytes(nonce); // 每次新生成，随密文一起传输
```

也不要靠截断字符串派生密钥——那会把密钥空间压缩到口令的熵，而非 128 位。
