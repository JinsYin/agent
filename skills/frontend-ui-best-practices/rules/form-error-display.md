---
title: 错误呈现三件套
impact: HIGH
tags: form, validation, error, accessibility
---

## 错误呈现三件套

字段校验失败时同时做三件事，缺一不可：

1. 输入框边框切换为错误态颜色
2. **紧贴该输入框下方**显示具体错误信息
3. 用户重新输入时**实时清除**该字段的错误态

只变边框颜色不给文案，用户知道错了但不知道为什么错；把错误信息汇总到表单顶部或 toast 里，用户还得自己对应到哪个字段。

错误文案要具体：说"手机号需为 11 位数字"，不说"格式不正确"。

```tsx
<Input aria-invalid={!!error} className={error && "border-destructive"} />
{error && <p className="text-sm text-destructive mt-1">{error.message}</p>}
```
