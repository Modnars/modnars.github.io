---
title: Go 语言高性能编程
date: 2024-05-20 11:33:00
abstract: Go 语言高性能编程 学习笔记
tags:
    - "Go"
categories:
    - "Notes"
---

# 常用数据结构

## 切片（slice）性能及陷阱

在 C 语言中，数组变量是指向第一个元素的指针，但是 Go 语言中并不是。Go 语言中，数组变量属于值类型(value type)，因此当一个数组变量被赋值或者传递时，实际上会复制整个数组。例如，将 a 赋值给 b，修改 a 中的元素并不会改变 b 中的元素：

```go
a := [...]int{1, 2, 3} // ... 会自动计算数组长度
b := a
a[0] = 100
fmt.Println(a, b) // [100 2 3] [1 2 3]
```

# 参考

[Go 语言高性能编程](https://geektutu.com/post/high-performance-go.html)
