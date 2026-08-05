---
title: Controller 方法名与排列顺序
impact: MEDIUM
tags: naming, controller, crud
---

## Controller 方法名与排列顺序

标准 CRUD 用固定动词，**不带资源名**（用 `create` 而非 `createUser`——类名已经说明了资源）：

`page`（分页）/ `list`（全量）/ `select`（下拉选项）/ `get` / `create` / `update` / `delete`（硬删）/ `remove`（软删）

非 CRUD 用业务动词 + 资源名：`cancelProcess`、`resetPassword`。

**排列顺序固定**：先全部 CRUD，再非 CRUD；CRUD 内部按 `page → list → select → get → create → update → delete/remove`。顺序固定后，翻任何一个 Controller 都能在相同位置找到相同种类的方法。

```java
@GetMapping                    public RPage<UserListItemResponse> page(...)
@GetMapping("/all")            public RList<UserListItemResponse> list(...)  // 需限制最大条数防 OOM
@GetMapping("/options")        public RList<UserOptionResponse> select()
@GetMapping("/{id}")           public R<UserDetailResponse> get(...)
@PostMapping                   public R<UserCreateResponse> create(...)
@PutMapping("/{id}")           public R<Boolean> update(...)
@DeleteMapping("/{id}")        public R<Boolean> remove(...)
```

同一个 Controller 里不允许同时存在 `delete` 和 `remove`——两个删除语义并存必然被误用。
