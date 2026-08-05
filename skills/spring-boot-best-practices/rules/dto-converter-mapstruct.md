---
title: 用 MapStruct 转换，不手写赋值
impact: MEDIUM
tags: dto, mapstruct, converter
---

## 用 MapStruct 转换，不手写赋值

Entity / Dto / Request / Response 之间的转换一律走 MapStruct。手写 `setXxx` 的问题是加字段时不会报错，只是静默丢值。

每个功能一个 `{Feature}Converter`，标 `@Mapper(componentModel = "spring")`：

```java
@Mapper(componentModel = "spring")
public interface UserConverter {
    UserDetailResponse toResponse(UserEntity entity);
    List<UserListItemResponse> toListItems(List<UserEntity> entities);
    UserEntity toEntity(UserCreateRequest request);
    void updateEntity(@MappingTarget UserEntity entity, UserUpdateRequest request);
}
```

实际用到的每种转换都要显式声明方法，不要在 Service 里临时拼装。
