# 一个整数的诞生

## 1. Swift的协议

Swift是一门面向协议的语言。在Swift中，协议，也就是`protocol`是一种接口，更准确地说，是表达了一种概念。比如`equatable`协议表达的概念就是可比较的，为了表达可比较的这种概念，用Swift的方式表达就是`func operator==()`和`func operator !=()`。

Swift 4.0标准库中定义了超过60个`protocol`，每个`protocol`都表达了特定的概念。特点如下：

1. Swift的`protocol`是可以继承的，比如`comparable`就继承自`equatable`，这里的隐含的概念就是说，一个类型如果符合`comparable`，那它一定是可比较相等性的。

2. 协议可以定义默认的实现，这是一个非常有用的特性，因为你有一个type，符合`comparable`, 你不需要写出全部六个方法，你只需要写出两个方法，

## 2. Swfit中的类型

在Swift中，类型是不能独立存在的，类型是为了表达一个或一组`protocol`，也就是概念。比如`Array`是一个`type`，但是这个type不是凭空出现的，它的意义在于实例化一组概念：

```
public struct Array<> : Collection, 

```

于是我们知道，Array是一个Collection，而且是一个RandomAccessCollection等。