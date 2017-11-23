[上一篇：一个整数的诞生](birth_of_int32.md)

# Equatable和Comparable

上次我们谈了Swift标准库是如何通过一系列`protocol`来定义一个整数的，今天我们进一步深入标准库源代码，看看这些`protocol`的真面目。

## 1. Equatable协议

我们从最简单的`Equatable`开始，这是Swift标准库最基本的协议之一：

```
// file: Equatable.swift

public protocol Equatable {
  static func == (lhs: Self, rhs: Self) -> Bool
}

extension Equatable {
  @_inlineable
  @_transparent
  public static func != (lhs: Self, rhs: Self) -> Bool {
    return !(lhs == rhs)
  }
}
```

`Equatable`表达的`concept`简单明了：如果一个`type`的实例可以比较相等性（equality），那这个`type`就是`equatable`，用Swift的语言表示就是

```
func ==(lhs:rhs:)
```


## 2. Comparable

`Comparable`表达的概念比`Equatable`更多了一层：除了比较相等性之外，还应该可以比较大小。

```
// file: Comparable.swift

public protocol Comparable : Equatable {
  static func < (lhs: Self, rhs: Self) -> Bool
  static func <= (lhs: Self, rhs: Self) -> Bool
  static func >= (lhs: Self, rhs: Self) -> Bool
  static func > (lhs: Self, rhs: Self) -> Bool
}
```
