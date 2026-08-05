---
title: Dto 不外露，双角色必须拆开
impact: HIGH
tags: dto, layering, api-contract
---

## Dto 不外露，双角色必须拆开

`{Name}Dto` 只用于 Service 之间、Service 内部方法之间，或 MQ 载荷。Controller **既不接收也不返回** Dto。

一个对象若同时承担「内部传输」和「接口返回」，必须拆成 Dto + Response 两个类。图省事合并的代价是：内部加一个字段就意外扩大了对外契约，且没有任何编译期提示。

```
Service ←→ Service     用 Dto
MQ 载荷                用 Dto
Controller ←→ 外部      用 Request / Response
```
