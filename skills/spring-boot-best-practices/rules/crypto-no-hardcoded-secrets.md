---
title: 密钥不入代码，按用途分离
impact: CRITICAL
impactDescription: 密钥进 git history 后无法真正删除
tags: crypto, secrets, key-management, rotation
---

## 密钥不入代码，按用途分离

私钥、对称密钥、IV、nonce、盐、生产 secret 一律不得硬编码。进了 git history 就等于永久泄漏——即使后续删除文件，历史提交里仍在。

- 加密、签名、MAC/完整性**各用独立密钥**，一把钥匙多用会让一处泄漏波及全部安全属性
- SM2 密钥用 PEM/DER，SM4 用 16 字节二进制
- 密文生命周期可能长于密钥轮换周期时，随载荷携带 `keyId`
- 二进制传输默认 Base64，除非对端协议要求 hex

没有 `keyId` 的历史密文在轮换后将无法解密，而这通常在轮换当天才被发现。
