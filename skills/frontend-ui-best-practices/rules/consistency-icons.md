---
title: 同功能操作使用同一图标
impact: LOW
tags: consistency, icon, ux
---

## 同功能操作使用同一图标

相同功能在全站必须用**同一个图标**（样式一致，且除非另有说明尺寸也一致）：新增、编辑、删除、复制、刷新、关闭弹层、搜索、禁用、发布/取消发布、密码显隐等。

图标是用户学一次就复用的视觉词汇。同一个"删除"在 A 页面是垃圾桶、B 页面是叉号，用户就得在每个页面重新学一遍。

建议把图标集中在一处导出，各页面引用而非各自挑选：

```tsx
// icons.ts —— 单一事实来源
export const ActionIcon = {
  create: Plus, edit: Pencil, delete: Trash2,
  copy: Copy, refresh: RotateCw, search: Search,
} as const;
```
