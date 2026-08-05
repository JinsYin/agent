---
title: 混合校验策略：blur 首验、change 复验、submit 兜底
impact: HIGH
impactDescription: 表单放弃率最高的成因
tags: form, validation, ux
---

## 混合校验策略：blur 首验、change 复验、submit 兜底

| 时机 | 行为 |
|---|---|
| `blur` | 首次校验该字段 |
| `change` | **仅在该字段已出错后**转为实时校验 |
| `submit` | 全字段兜底校验 |
| `change` | 辅助反馈（如密码强度）直接实时 |

两个极端都不可取：只在 submit 校验，用户填完一长串才知道第一项就错了；从一开始就 change 实时校验，则用户刚敲第一个字符就看到"格式不正确"，属于对着还没写完的输入报错。

先 blur 后 change 的混合策略避开了这两点——出错前不打扰，出错后即时确认修正是否生效。

```tsx
useForm({ mode: "onBlur", reValidateMode: "onChange" })
```
