---
title: BouncyCastle Provider 启动时注册一次
impact: HIGH
tags: crypto, bouncycastle, provider, startup
---

## BouncyCastle Provider 注册一次

优先用 `org.bouncycastle:bcprov-jdk18on`，在应用启动时注册一次，不要在每次加解密时重复注册——`addProvider` 有同步开销，热路径上反复调用会成为瓶颈。

**正确：**

```java
@Configuration
public class CryptoConfig {
    @PostConstruct
    public void registerProvider() {
        if (Security.getProvider(BouncyCastleProvider.PROVIDER_NAME) == null) {
            Security.addProvider(new BouncyCastleProvider());
        }
    }
}
```

模块若已使用其他合规 provider，沿用即可，不要混装两个。
