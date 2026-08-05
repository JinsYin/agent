---
title: 同一资源的弹层尺寸与遮罩保持一致
impact: LOW
tags: overlay, modal, drawer, consistency
---

## 同一资源的弹层尺寸与遮罩保持一致

- 同一资源的**新建 / 编辑 / 查看**弹层，宽高保持一致
- 抽屉外的遮罩色与不透明度，与弹窗外的遮罩保持一致

尺寸跳变会让用户在切换操作时重新定位视线；遮罩不一致则会让人误以为进入了不同层级的界面。

```tsx
// 三种模式共用同一套尺寸常量
const DIALOG_SIZE = "sm:max-w-2xl min-h-[520px]";

<DialogContent className={DIALOG_SIZE}>  {/* 新建 */}
<DialogContent className={DIALOG_SIZE}>  {/* 编辑 */}
<DialogContent className={DIALOG_SIZE}>  {/* 查看 */}
```
