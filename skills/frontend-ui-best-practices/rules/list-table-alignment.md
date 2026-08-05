---
title: 表头与单元格一律左对齐，空值显示短横线
impact: MEDIUM
tags: list, table, alignment, empty-state
---

## 表头与单元格一律左对齐，空值显示短横线

- 表头与单元格内容全部**左对齐**，单元格内不加额外左内边距
- 空单元格与详情页缺失值显示 `-`，不留空白

留空白无法区分三种情况：数据为空、加载失败、还是渲染漏了。统一显示 `-` 明确表示"此处确实没有值"。

表头与内容对齐方式不一致（表头居中、内容左对齐）会让列边界在视觉上错位，扫读时容易串行。

```tsx
<TableHead className="text-left">机构名称</TableHead>
<TableCell className="text-left">{org.name || "-"}</TableCell>
```

数字列如需右对齐以便对位比较，表头须同步右对齐。
