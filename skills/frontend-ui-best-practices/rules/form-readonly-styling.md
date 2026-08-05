---
title: 不可编辑字段灰化文字而非整个输入框
impact: MEDIUM
tags: form, readonly, disabled, ux
---

## 不可编辑字段灰化文字而非整个输入框

字段不可编辑时，灰化的是**输入框内的文字**，不是输入框本身；悬停时显示 `not-allowed` 光标。

整体灰化会让输入框在视觉上退化成背景，用户往往看不清里面的值——而这些值通常恰恰是需要被读取的（如系统生成的编号、当前所属机构）。

```tsx
<Input
  readOnly
  value={code}
  className="text-muted-foreground cursor-not-allowed"
/>
```

同理，密码框应带眼睛图标，点击在明文与掩码间切换——让用户能确认自己输入的内容，尤其是在没有"确认密码"字段时。
