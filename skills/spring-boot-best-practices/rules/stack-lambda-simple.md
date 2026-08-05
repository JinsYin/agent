---
title: 避免复杂 lambda 表达式
impact: LOW
impactDescription: 栈轨迹只显示 lambda$xxx$0，异常定位困难
tags: stack, java, lambda, readability
---

## 避免复杂 lambda 表达式

多行、嵌套或带副作用的 lambda 在异常栈轨迹里只显示为 `lambda$methodName$0`——这个名字既不表达意图，也不指向具体哪一行。方法里若有多个 lambda，序号还会随代码顺序变动，跨版本对比栈轨迹时对不上。

抽成具名方法后，栈轨迹直接指向出错处，方法名本身也说明了在做什么。

**错误（异常只会报在 `lambda$process$0`，看不出是哪一步失败）：**

```java
list.stream()
    .map(item -> {
        var dto = new ItemDTO();
        dto.setName(item.getName());
        if (item.getParent() != null) {
            dto.setParentName(item.getParent().getName());
        }
        return dto;
    })
    .toList();
```

**正确（栈轨迹指向 `toDTO`，且该方法可单独测试与复用）：**

```java
list.stream()
    .map(this::toDTO)
    .toList();

private ItemDTO toDTO(Item item) {
    var dto = new ItemDTO();
    dto.setName(item.getName());
    if (item.getParent() != null) {
        dto.setParentName(item.getParent().getName());
    }
    return dto;
}
```

**判据**：超过一行就提取。单表达式 lambda（`x -> x.getId()`、`x -> x > 0`）保持内联，方法引用优先于等价的 lambda。
