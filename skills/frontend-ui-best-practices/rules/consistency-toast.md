---
title: Toast 必须图标 + 文字，且按级别区分颜色
impact: MEDIUM
tags: consistency, toast, feedback, accessibility
---

## Toast 必须图标 + 文字，且按级别区分颜色

Toast 同时给出**图标和文字**，图标颜色按严重级别变化（成功 / 警告 / 错误 / 信息）。

只有文字会让用户必须读完才知道是成功还是失败；只靠颜色区分则对色觉障碍用户无效——图标形状提供了不依赖颜色的第二重信号，这是可访问性的基本要求，不是装饰。

```tsx
toast.success("保存成功", { icon: <CheckCircle className="text-green-600" /> });
toast.error("保存失败：机构编码已存在", { icon: <XCircle className="text-destructive" /> });
```

错误 toast 要给出**具体原因**，不要只说"操作失败"。
