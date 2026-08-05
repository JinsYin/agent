---
title: 中文与西文、行内代码之间加空格
impact: LOW
tags: cjk, typography, spacing
---

## 中文与西文、行内代码之间加空格

与数字同理，中文与拉丁字母、行内代码之间也需要半角空格分隔。行内代码尤其明显——反引号渲染成带背景色的方块后，紧贴中文会显得挤压。

**错误：**

```markdown
用Kustomize管理overlay，执行`kubectl apply`即可。
```

**正确：**

```markdown
用 Kustomize 管理 overlay，执行 `kubectl apply` 即可。
```

**标点旁不加**：中文标点已含空白，`执行 `kubectl apply` 即可。` 中句号前不再补空格。

这条是 LOW，因为它不影响理解。但在同一份文档里时加时不加，比统一不加更显得潦草——**一致性比选哪一种更重要**。
