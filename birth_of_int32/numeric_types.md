[上一篇：Equatable和Comparable](equatable_and_comparable.md)

# Numeric Types

整数除了可以比较大小，还可以进行加减乘除等运算。Swift为了表达这一概念，定义了一系列和数值计算有关的协议，今天我们就来谈谈这些协议。我们从最基本的`Numeric`开始。

## Numeric protocol

`Numeric`协议定义了一系列适用于整数和浮点数的运算，比如相加，相减，互乘等。

```
public protocol Numeric : Equatable, ExpressibleByIntegerLiteral {
  init?<T : BinaryInteger>(exactly source: T)

  associatedtype Magnitude : Comparable, Numeric
  var magnitude: Magnitude { get }
  
  static func +(_ lhs: Self, _ rhs: Self) -> Self
  static func +=(_ lhs: inout Self, rhs: Self)
  static func -(_ lhs: Self, _ rhs: Self) -> Self
  static func -=(_ lhs: inout Self, rhs: Self)
  static func *(_ lhs: Self, _ rhs: Self) -> Self
  static func *=(_ lhs: inout Self, rhs: Self)
}
```

可以看到，`Numeric`协议只定义`+`、`-`和`*`运算，除法运算并不包括在其中，我们会看到除法运算定义在其它的协议中。

对每一个协议，标准库通常都会以`extension`的方式提供默认的实现，但是`Numeric`协议除外，标准库并没有为任何一个方法提供默认实现。也就是说，你最好也不要直接使用`Numeric`协议。

不建议使用`Numeric`，那使用什么呢？

## SignedNumeric

```
public protocol SignedNumeric: Numeric {
  static prefix func -(_ operand: Self) -> Self
  mutating func negate()
}

extension SugnedNumeric {
  public static prefix func -(_ operand: Self) -> Self {
    var result = operand
    result.negate()
    return result
  }
}
```

`SginedNumeric`在`Numeric`的基础上定义了`func -()`





