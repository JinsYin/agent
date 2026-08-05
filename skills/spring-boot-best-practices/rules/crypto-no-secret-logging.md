---
title: 密钥与明文不进日志
impact: CRITICAL
impactDescription: 日志系统通常权限更宽
tags: crypto, logging, secrets, error-handling
---

## 密钥与明文不进日志

**禁止**记录明文、私钥、对称密钥、完整密文、完整签名、解密后载荷。日志往往被集中采集、长期留存，且访问权限比数据库宽得多——写进日志等于扩大了泄漏面。

解密失败、认证标签不匹配、密文格式非法、验签失败一律按**安全错误**处理：返回稳定的业务错误码，不要把密码学细节透给调用方（那等于给攻击者提供预言机）。

**错误：**

```java
log.error("解密失败, ciphertext={}, key={}", cipherText, keyHex, e); // ❌
```

**正确：**

```java
log.warn("解密失败, keyId={}, len={}", keyId, cipherText.length);
throw new BizException(ErrorCode.DECRYPT_FAILED);
```

测试需覆盖：SM2 `C1C3C2` 加解密往返、签名验证的成功与失败、SM3 已知答案摘要、SM4 nonce 唯一性与篡改检测。
