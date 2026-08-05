---
title: 类名后缀对照表
impact: MEDIUM
tags: naming, class
---

## 类名后缀对照表

| 种类 | 模式 | 示例 | 位置 |
|---|---|---|---|
| 启动类 | `{Project}Application` | `OrderServerApplication` | 根包 |
| Controller | `{Name}Controller` | `OrderController` | `controller/` |
| Service 接口 | `{Name}Service` | `OrderService` | `service/` |
| Service 实现 | `{Name}ServiceImpl` | `OrderServiceImpl` | `service/impl/` |
| Mapper | `{Name}Mapper` | `OrderMapper` | `mapper/` |
| Entity | `{Name}Entity` | `OrderEntity` | `entity/` |
| 请求体 | `{Name}{Action}Request` | `OrderCreateRequest` | `dto/{业务}/` |
| 响应体 | `{Name}{View}Response` | `OrderDetailResponse` | `dto/{业务}/` |
| 查询参数 | `{Name}Query` / `{Name}PageQuery` | `OrderPageQuery` | `dto/{业务}/` |
| 内部传输 | `{Name}Dto` | `OrderSummaryDto` | `dto/{业务}/` |
| 异常 | `{Name}Exception` | `OrderLockedException` | `exception/` |
| MapStruct | `{Name}Converter` | `OrderConverter` | `converter/` |
| 配置 | `{Name}Config` / `{Name}Properties` | `RedisConfig` | `config/` |
| 枚举 | 名词 | `OrderStatus` | `enums/` |
| 常量 | `{Name}Constants` | `AuthConstants` | `constants/` |
| 工具 | `{Name}Utils` | `DateUtils` | `utils/` |

`{Name}` 是业务名词且**用单数**（`User` 而非 `Users`）；动词放方法名，不进类名（不要 `OrderCreator`）。除行业通用缩写外不缩写（`Order` 不写 `Ord`）。
