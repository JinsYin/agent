---
title: 关闭浏览器自动填充
impact: MEDIUM
tags: form, autocomplete, security
---

## 关闭浏览器自动填充

- 表单与输入框设 `autocomplete="off"`
- 密码类输入设 `autocomplete="new-password"`

后台管理系统里浏览器的自动填充经常张冠李戴——把登录密码填进"新增用户"的密码框，或把个人邮箱填进客户邮箱字段，用户不检查就直接提交。

密码框必须用 `new-password` 而不是 `off`：主流浏览器对密码字段会忽略 `off`，只有 `new-password` 才真正生效。

```tsx
<form autoComplete="off">
  <Input name="email" autoComplete="off" />
  <Input name="password" type="password" autoComplete="new-password" />
</form>
```
