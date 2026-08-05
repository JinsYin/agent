---
title: 弹层关闭后必须重置状态
impact: CRITICAL
impactDescription: 脏数据被误提交
tags: overlay, modal, drawer, form, state
---

## 弹层关闭后必须重置状态

新建/编辑弹窗或抽屉关闭后再次打开，必须回到干净状态：清空输入值、清除校验错误、重置为初始默认值。

残留状态的后果不只是观感问题——用户以为在新建，实际带着上一条记录的字段提交了。这类 bug 在手工测试里很难复现，因为测试者通常刷新页面而不是连续开关弹层。

**错误（组件常驻，状态跨次数保留）：**

```tsx
const [open, setOpen] = useState(false);
const form = useForm({ defaultValues });
// ❌ 关闭只是隐藏，form 状态还在
```

**正确（关闭时显式重置，或用 key 强制重挂载）：**

```tsx
<Dialog open={open} onOpenChange={(v) => { setOpen(v); if (!v) form.reset(defaultValues); }}>

// 或
{open && <UserFormDialog key={editingId ?? "new"} />}
```
