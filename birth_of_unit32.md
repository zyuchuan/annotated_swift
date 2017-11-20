# 一个整数的诞生

>*现在一切都清楚了：一个那天早些时候去过思里博尔的人；一个熟悉艾克罗伊德并知道他买了一台口述录音机的人；一个懂得机械原理的人；一个有机会在弗洛拉小姐到来前从银柜拿走剑的人；一个拿着装得下口述录音机的容器（比如一只黑包）的人；一个在帕克给警察打电话时能单独在书房里呆几分钟的人。事实上这个人就是——谢泼德医生！*
>
> —— 赫尔克里·波洛 

## 1. Swift的协议

Swift是一门面向协议的语言。在Swift中，协议，也就是`protocol`是一种接口，更准确地说，是表达了一种概念。比如`equatable`协议表达的概念就是可比较的，为了表达可比较的这种概念，用Swift的方式表达就是`func operator==()`和`func operator !=()`。

Swift 4.0标准库中定义了超过60个`protocol`，每个`protocol`都表达了特定的概念。特点如下：

1. Swift的`protocol`是可以继承的，比如`comparable`就继承自`equatable`，这里的隐含的概念就是说，一个类型如果符合`comparable`，那它一定是可比较相等性的。

2. 协议可以定义默认的实现，这是一个非常有用的特性，因为你有一个type，符合`comparable`, 你不需要写出全部六个方法，你只需要写出两个方法，

## 2. Swift中的类型

在Swift中，类型是不能独立存在的，类型是为了表达一个或一组`protocol`，也就是概念。比如`Array`是一个`type`，但是这个type不是凭空出现的，它的意义在于实例化一组概念：

```
public struct Array<> : Collection, 

```

于是我们知道，Array是一个Collection，而且是一个RandomAccessCollection等。

## 3. Int32的诞生

说了那么多，下面我们通过一个例子，来说明Swift中，协议和类型是如何相互作用的。我们就以`Int32`为例，下面这个图是

![](/assets/Int32_hierarchy.png)

可以看到，即使是`Int32`这样一个简单的整数类型，也有很复杂的继承关系，表达了相当多的概念。下面来逐一分析。

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


