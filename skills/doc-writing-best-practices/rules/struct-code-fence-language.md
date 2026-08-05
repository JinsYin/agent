---
title: 代码块必须标注语言
impact: MEDIUM
impactDescription: 无标注则不高亮，长代码块的可读性大幅下降
tags: struct, markdown, code-block
---

## 代码块必须标注语言

不带语言标注的围栏代码块不会被高亮，几十行的配置或代码就变成一片等宽灰字。标注的成本是几个字符。

**错误：**

````markdown
```
FROM node:24-alpine
RUN corepack enable
```
````

**正确：**

````markdown
```dockerfile
FROM node:24-alpine
RUN corepack enable
```
````

常见标注：`bash` `java` `tsx` `ts` `sql` `yaml` `dockerfile` `json` `markdown` `diff`。

**标注要准确**：把 YAML 标成 `json` 比不标更糟——高亮会按错误的语法着色，读者据此判断结构就会出错。

**纯输出用 `text`**：终端输出、日志片段、目录树没有对应语言时显式标 `text`，表明"这里确实不需要高亮"，而不是忘了标。
