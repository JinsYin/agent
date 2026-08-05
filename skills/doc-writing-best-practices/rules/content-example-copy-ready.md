---
title: 示例要能直接抄用，不用占位符敷衍
impact: HIGH
impactDescription: 需要读者自行补全的示例会被抄错，且错误发生在他们的环境里
tags: content, example
---

## 示例要能直接抄用，不用占位符敷衍

`foo` / `bar` / `<your-value-here>` 这类示例把补全的责任推给读者。补全需要的恰恰是文档没写的那部分知识——于是要么抄错，要么回来问。

**弱：**

```bash
docker build -f <dockerfile> --build-arg <key>=<value> -t <tag> .
```

**强（用项目真实会出现的值，可直接粘贴执行）：**

```bash
docker build -f deploy/docker/Dockerfile.backend \
  --build-arg MODULE=dap-admin \
  -t dap-admin:latest .
```

确实需要读者替换的部分，用**语义明确的大写占位**并紧跟一行说明来源：

```bash
export NEXUS_PASSWORD=<你的 Nexus 密码，见密码管理器 "内网 Nexus" 条目>
```

**错误示例必须标注错在哪。** 只贴一段"错误写法"而不说明为什么错，读者只会记住形状、记不住原因，下次换个形状照犯。
