### 3.1 Equatable协议

`Equatable`协议是Swift标准库中一个非常基础的协议，任何一个可以比较相等性的类型都是`Equatable`，`Equatable`的定义相当简单：

```
// file: Equatable.swift

public protocol Equatable {
    static func ==(lhs: Self, rhs: Self) -> Bool
}

extension Equatable {
    @_inlineable
    @_transparent
    public static func !=(lhs: Self, rhs: Self) -> Bool {
        return !(lhs == rhs)
    }
```

可以看到，Equatable协议实际只包含一个方法`func ==(lhs:rhs:)`，另一个方法`func !=(lhs:rhs:)`标准库已经有了默认的实现，不需要我们再写。

### 3.2 Comparable

`Comparable`协议定义了一种有序性的概念，也就是说可以比较大小，确切地说，可以用`<`和`==`操作符比较顺序。

```
// file: Comparable.swift

public protocol Comparable: Equatable {
    static func <(lhs: Self, rhs: Self) -> Bool 
    static func <=(lhs: Self, rhs: Self) -> Bool
    static func >=(lhs: Self, rhs: Self) -> Bool
    static func >(lhs: Self, rhs: Self) -> Bool
}
```

同样，如果你有一个`type`遵从`Comparable`协议，你不需要写出上面全部的方法，标注库已经贴心地为你提供了默认的实现，你只需要写`func==(lhs:rhs:)`和`func<(lhs:rhs:)`就好。

```
// file: Comparable.swift

extension Comparable {
    @_inlineable
    public static func >(lsh: Self, rhs: Self) -> Bool {
        return rhs < lhs
    }
    
    @_inlineable
    public static func <=(lhs: Self, rhs: Self) -> Bool {
        return !(rhs < lhs)
    }
    
    @_inlineable
    public static func >=(lhs: Self, rhs: Self) -> Bool {
        return !(lhs < rhs)
    }
}
```

### 3.3 Numeric

`Numeric`协议定义了可以作用于标量上的一系列操作，比如`+`, `-`等。

```
public protocol Numeric : Equatable, ExpressibleByIntegerLiteral {
    
    static func +(_ lhs: Self, _ rhs: Self) -> Self
    static func +=(_ lhs: inout Self, rhs: Self)
    static func -(_ lhs: Self, _ rhs: Self) -> Self
    static func -=(_ lhs: inout Self, rhs: Self)
    static func *(_ lhs: Self, _ rhs: Self) -> Self
    static func *=(_ lhs: inout Self, rhs: Self)
}
```

说明：

1. 除法不是`Numeric`协议的一部分
2. `Numeric`协议不能作用于不同类型，这就是你不能写出`2+2.0`的原因

### 3.4 SignedNumeric

`SignedNumeric`定义了一个可以是正值，也可以是负值的类型。

`SignedNumeric`协议扩展了`Numeric`协议的操作。以执行反向操作。

`SignedNumeric`协议对所有的操作都有默认实现

```
public protocol SignedNumeric: Numeric {
    // The negation operator (prefix '-`) returns the additive inverse of
    // its argument.
    static prefix func -(_ operand: Self) -> Self
    
    mutating func negate()
}

extension SignedNumeric {
    @_inlineable
    @_transparent
    public static prefix func -(_ operand: Self) -> Self {
        var result = operand
        result.negate()
        return result
    }
    
    @_inline
    @_transparent
    public mutating func negate() {
        self = 0 - self
    }
}
```

### 3.5 BinaryInteger

Concept: 一个可以用二进制表示的整数。

`BinaryInteger`协议是Swift标准库中定义的所有整数类型的基础协议，所有整数类型，如`Int`, `UInt32`等，都遵从这个协议。

```
public protocol BinaryInteger: 
    Hashable, Numeric, CustomStringConvertible, Strideable
    where Magnitude: BinaryInteger, Magnitude.Magnitude == Magnitude {
    
    static var isSigned: Bool { get }
    
    // Creates an integer from the given floating-point value.
    // If the value passed as 'source` is not representable exactly, the
    // result is nil.
    init?<T: BinaryFloatingPoint>(exactly source: T)

    // Creates an integer from the given floating-point value, rounding
    // toward zero.
    init<T: BinaryFloatingPoint>(_ source: T)
    
    // Creates a new instance from the given integer.
    init<T: BinaryInteger>(_ source: T)

    ...
    
    static func /(_ lhs: Self, _ rhs: Self) -> Self
    static func /=(_ lhs: inout Self, _ rhs: Self)
    static func %(_ lhs: Self, _ rhs: Self) -> Self
    static func %=(_ lhs: inout Self, _ rhs: Self)
    
    ...
}
```

`BinaryInteger`的源代码很长，由于篇幅的原因，就不一一列举了。总的来说，定义了一下的操作：

1. 不同数值类型之间的转换：你可以从一个浮点型数值，或其它任意类型的`binary integer`创建出一个遵从`BinaryInteger`协议的新实例。`BinaryInteger`协议定义了四种不同的创建实例的方法。
2. 整形数值之间的比较：你可以使用关系运算符，如`<`，`==`等，比较不同类型的`binary integer`。 


### 3.6 FixedWidthInteger

Concept: An integer type that uses a fixed size for every instance.

`FixedWidthInteger`协议在`BinaryInteger`的基础上增加了位操作，位偏移，以及溢出处理等操作。

### 3.7 SignedInteger

Concept: An integer type that can represent both positive and negative values.

### 3.8 ExpressibleByIntegerLiteral

Concept: 一个可以用整形字面量（integer literal）初始化的类型。

标准库中的整形和浮点类型，比如`Int`和`Double`，都遵从`ExpressibleByIntegeralLiteral`协议，也就是说你可以通过赋值的方法给变量初始化。

```
// Type inferred as Int
let cookieCount = 12

// An array of Int
let chipsPerCookie = [1, 2, 3, 5]

// A floating-point value initialized using an integer literal
let redPercentage: Double = 1
// redPercentage = 1.0
```

这个协议的定义相当简单：

```
file: CompilerProtocols.swift

public protocol ExpressibleByIntegerLiteral {
    associatedtype IntegerLiteralTYpe: _ExpressibleByBuiltinIntegerLiteral

    init(integerLiteral value: IntegerLiteralType)
}
```

当让，你不需要自己实现一个`init`方法，应为标准库已经替你实现了一个：

```
// file: Integers.swift

extension ExpressibleByIntegerLiteral where Self: _ExpressibleByBuiltinIntegerLiteral {
    @_inlineable
    @_transparent
    public init(integerLiteral value: Self) {
        self = value
    }
}
```

### 3.9 CustomStringConvertible

这是一个使用很普遍的协议，这个协议定义了一个方法，可以将一个实例转化成一个字符串。

这个协议很简单，只有一个`description`属性。比如说

```
file: OutputStream.swift

public protocol CustomStringConvertible {
    var description: String { get }
}
```
下面举例说明其用法

```
struct Point {
    ...
}
```
你希望可以打印出

``` 
extension Point: CustomStringConvertible {
    var description: String {
        return "(\(x), \(y))"
    } 
}
```