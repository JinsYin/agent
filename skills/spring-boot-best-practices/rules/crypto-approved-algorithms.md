---
title: 只用批准的国密算法与模式
impact: CRITICAL
impactDescription: 弱算法/ECB 直接构成可利用漏洞
tags: crypto, sm2, sm3, sm4, algorithms
---

## 只用批准的国密算法与模式

| 用途 | 必须 | 禁止 |
|---|---|---|
| 非对称加密 | SM2 `C1C3C2` | `C1C2C3`（仅隔离的兼容层可留） |
| 签名 | SM2 + SM3 摘要 | — |
| 摘要 | SM3 | MD5、SHA1 |
| 对称加密 | `SM4/GCM/NoPadding` | **任何 `SM4/ECB/*`** |
| 兼容场景 | `SM4/CBC/PKCS5Padding` | DES、3DES、RC4 |

ECB 对相同明文块产生相同密文块，结构直接从密文可见——这不是强度问题，是模式本身不提供语义安全。

同样禁止自己实现密码学原语。需要什么就用 Bouncy Castle 已有的实现。
