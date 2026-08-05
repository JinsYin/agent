---
title: 破坏性操作用自定义确认弹窗
impact: CRITICAL
impactDescription: 误删不可恢复
tags: overlay, modal, destructive, confirm
---

## 破坏性操作用自定义确认弹窗

删除、禁用、批量操作等不可逆动作必须弹确认框，且**不能用原生 `alert` / `confirm`**。原生弹窗无法说明将要删除什么、影响多少条，样式也不受控，用户往往条件反射点确定。

**错误：**

```tsx
if (confirm("确定删除？")) deleteUser(id); // ❌ 说不清删的是谁
```

**正确（说清对象与后果）：**

```tsx
<AlertDialog>
  <AlertDialogContent>
    <AlertDialogTitle>删除用户「{user.name}」？</AlertDialogTitle>
    <AlertDialogDescription>
      该用户的 {user.recordCount} 条授权记录将一并移除，此操作不可恢复。
    </AlertDialogDescription>
    <AlertDialogAction variant="destructive">删除</AlertDialogAction>
    <AlertDialogCancel>取消</AlertDialogCancel>
  </AlertDialogContent>
</AlertDialog>
```

确认按钮用破坏性配色，且**不要**设为默认聚焦项——回车不应该能直接确认删除。
