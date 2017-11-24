# 神秘的Builtin模块

*原文：[Swift's mysterious Builtin module](http://ankit.im/swift/2016/01/12/swift-mysterious-builtin-module/)*

<hr/>

当你在Playground中cmd+click某个量，比如`Int`的时候，你可能会看到下面的代码：

```
/// A 64-bit signed integer value type
public struct Int : SignedIntegerType, Comparable, Equatable, {
    public var value: Builtin.Int64
    ...
}
```

或者当你阅读Swift标准库源代码的时候，你可能会看到许多形如`Builtin.*`的方法，就像下面这些：

* `Builtin.Int1`
* `Builtin.RawPointer`
* `Builtin.NativeObject`
* `Builtin.allocRaw(size._builtinWordValue,Builtin.alignof(Memory.self)))`

究竟什么是`Builtin`？

## Clang, Swift Compiler, SIL, IR, LLVM

要弄明白`Builtin`是什么，先要明白Objective-C和Swift编译器是怎样工作的。

![How Objective-C compilers works](/assets/fig_01.png)

上图显示了Objective-C代码的编译过程：Objective-C代码经过Clang处理后，生成**LLVM Intermediate Representation (IR)**，再经过LLVM处理，最后生成机器码。

可以把LLVM IR想象成某种高级的汇编语言，这种汇编语言和CPU架构（如i386或ARM）无关。任何一个编译器，只要能生成LLVM IR，就可以用LLVM生成和特定的CPU架构兼容的机器码。

明白了这点，我们再来看Swift编译器是如何工作的：

![How Swift compilers works](/assets/fig_02.png)

Swift代码首先被编译成SIL (Swift Intermediate Represention)，然后再被编译成LLVM IR进入LLVM编译器，最后生成机器码。而SIL无非就是LLVM IR的一层Swift外壳（swifty wrapper），我们有很多理由需要SIL：比如确保变量在使用之前被初始化、检测不可执行的代码（unreachable code），优化代码等。如果你想知道SIL具体干了些啥，可以去看看这个[视频](https://www.youtube.com/watch?v=Ntj8ab-5cvE)。

我们再来看看LLVM IR，对于下面的代码：

```
let a = 5
let b = 6
let c = a + b
```

我们可以用这个命令

```
swiftc -emit-ir addswift.swift
```
将这三条语句编译成LLVM IR代码（以`//^`开头的语句为注释）：

```
...
//^ store 5 in a
store i64 5, i64* getelementptr inbounds (%Si* @_Tv8addswift1aSi, i32 0, i32 0), align 8
  
//^ store 6 in b
store i64 6, i64* getelementptr inbounds (%Si* @_Tv8addswift1bSi, i32 0, i32 0), align 8
  
//^ load a to virtual register %5
%5 = load i64* getelementptr inbounds (%Si* @_Tv8addswift1aSi, i32 0, i32 0), align 8

//^ load b to virtual register %6
%6 = load i64* getelementptr inbounds (%Si* @_Tv8addswift1bSi, i32 0, i32 0), align 8

//^ call llvm's signed addition with overflow on %5 and %6
//^ returns two values: sum and a flag if overflowed
%7 = call { i64, i1 } @llvm.sadd.with.overflow.i64(i64 %5, i64 %6)

//^ extract first value to %8
%8 = extractvalue { i64, i1 } %7, 0 
  
//^ extract second value to %9
%9 = extractvalue { i64, i1 } %7, 1

//^ if overflowed jump to trap otherwise jump to label 10  
br i1 %9, label %11, label %10
  

; <label>:10; preds = %once_done
//^ store result in c
store i64 %8, i64* getelementptr inbounds (%Si* @_Tv8addswift1cSi, i32 0, i32 0), align 8
  
ret i32 0
```

我知道你觉得上面的代码就像一坨屎，不过还是请注意：

* `i64`是定义在LLVM中的一个类型，代表64位整数。
* `llvm.sadd.with.overflow.i64`是个方法，这个方法将两个`i64`整数相加并返回两个值：一个表示相加的和，另一个标识操作是否成功。

## Builtin

我们已经知道在Swift中，`Int`实际上一个`struct`，而`+`是一个global的方法，这个方法被重载了，Int。严格说，`Int`和`+`不是Swift语言的一部分，它们是Swift标准库的一部分，也就是说它们不是Swift的原生的构造，那是不是意味着Swift很慢呢？当然不是。

这正是`Builtin`大展身手的地方。`Builtin`模块暴露了LLVM IR的`type`和`method`向标准库，这就意味着没有运行时查表的负担，这让在`Int`上施行的操作就像

`Int`中只有一个变量`value`，这个变量的类型是`Builtin.Int64`，



