---
title: golang 学习笔记
date: 2024-09-23 11:40:11
abstract: "golang 学习备忘录"
tags:
    - "Go"
categories:
    - "Notes"
---

### `any`（`interface{}`）的内存分配

`any` （也就是 `interface{}`）在 Go 中的内存分布，实际上包括两个指针字段，可以认为其定义如下：

```go
type emptyInterface struct {
    typ  *rtype
    word unsafe.Pointer
}
```

- `typ` 是一个指向类型信息的指针，描述了存储值的类型。
- `word` 是一个指向实际数据的指针。

当一个 `interface{}` 类型的变量存储一个值时，Go 运行时会根据值的类型和大小来分配内存。具体来说：

- **小对象**：如果存储的值是一个小对象（通常是指针大小或更小），数据可以直接存储在 `word` 字段中。
- **大对象**：如果存储的值是一个大对象，数据会存储在堆上，`word` 字段会指向这个堆上的数据。

在 64 位机器上，一个指针占 8 字节，所以一个 any 类型往往需要占 16 个字节。

### `type a = string` 和 `type a string`

`type a = string` 语义是使用 `a` 作为 `string` 类型的别名，本质上二者是同一类型，因此所有需要 `string` 类型参数的地方，都可以用 `a` 类型的变量，反之亦然。

`type a string` 语义则是定义一个类型 `a`，其底层类型是 `string`。二者不能直接赋值使用，因为编译器将他们视为两种独立的类型。对于这两种类型的参数需要使用显式的类型转换来实现类型转换。比如需要 `string` 类型的地方可以对 `a` 类型的变量 aa 使用 `string(aa)` 来实现类型转换。
