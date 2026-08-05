---
title: 占位值要显眼到不可能被当成真值
impact: HIGH
impactDescription: 看起来像真值的占位会被带上生产，且启动不报错
tags: secret, placeholder, config
---

## 占位值要显眼到不可能被当成真值

模板里的占位若写成 `password123`、`test-key`、`changeme`，它们看起来"像个值"——部署时容易被整段跳过，最终带上生产。而这类值通常不会导致启动失败，只会让加密、签名、鉴权在**看似正常**的情况下形同虚设。

占位要满足两条：一眼看出不是真值，且能被 grep 出来。

**错误：**

```yaml
DAP_CRYPTO_SM4_KEY: "0123456789abcdef"    # 像个合法密钥
DB_PASSWORD: "password"
```

**正确：**

```yaml
DAP_CRYPTO_SM4_KEY: "CHANGE_ME"
DB_PASSWORD: "CHANGE_ME"
```

上线前用一条命令兜底：

```bash
grep -rn 'CHANGE_ME' deploy/ && echo "❌ 存在未替换占位" && exit 1
```

这条检查应进发布清单（go-live checklist），而不是靠人记得。

**部分预填是合理的**：本地开发用的固定弱密码可以预填真值，但要就地注释说明"仅本地"，并确保该文件不用于其他环境。
