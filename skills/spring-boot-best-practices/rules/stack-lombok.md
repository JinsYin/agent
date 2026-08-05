---
title: Lombok 注解约定
impact: LOW
tags: stack, lombok, di
---

## Lombok 注解约定

| 场景 | 注解 |
|---|---|
| DTO | `@Data` + `@NoArgsConstructor` |
| Spring bean | `@RequiredArgsConstructor` + `final` 字段做构造注入 |
| 日志 | `@Slf4j`（禁止手写 `LoggerFactory.getLogger`） |
| Enum 字段 | `@Getter` |

构造注入优于 `@Autowired` 字段注入：`final` 字段保证依赖不可变，且缺依赖时在启动期就失败，而不是运行到那行才 NPE。

**保留显式实现**的两种情况：工具类（私有构造 + 全静态方法）、构造函数含参数校验的类——这两种情况 Lombok 生成的构造器反而会绕过约束。
