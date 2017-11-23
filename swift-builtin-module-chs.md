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

要弄明白`Builtin`是什么，先要明白Objective-C和Swift编译器是怎样工作的

![How Objective-C compilers works](/assets/fig_01.png)

如上图所示，Objective-C代码经过Clang处理后，生成**LLVM Intermediate Representation (IR)**，再经过LLVM处理，最后生成机器码。

可以把LLVM IR想象成某种高级的汇编语言，这种汇编语言不依赖于特定的架构，如i386或ARM等。任何一个编译器，只要能生成LLVM IR，就可以用LLVM生成和特定的CPU架构兼容的机器码。

明白了这点，我们再来看Swift编译器是如何工作的：

![How Swift compilers works](/assets/fig_02.png)

Swift代码首先被编译成SIL (Swift Intermediate Represention)，然后再被编译成LLVM IR进入LLVM编译器，最后生成机器码。

看到这里你大概已经猜到了SIL就是LLVM IR的一层外包装，我们有很多理由需要SIL：比如确保变量在使用之前被初始化，检测不可执行的代码，优化代码等。你可以看这个[视频](https://www.youtube.com/watch?v=Ntj8ab-5cvE)，如果你想知道SIL具体干了些啥。

## Builtin

我们已经知道在Swift中，`Int`实际上一个`struct`，而`+`是一个global的方法，这个方法被重载了，Int。严格说，`Int`和`+`不是Swift语言的一部分，它们是Swift标准库的一部分，也就是说它们不是Swift的原生的构造，那是不是意味着Swift很慢呢？当然不是。

这正是`Builtin`大展身手的地方。`Builtin`模块暴露了LLVM IR的`type`和`method`向标准库，这就意味着没有运行时查表的负担，这让在`Int`上施行的操作就像

`Int`中只有一个变量`value`，这个变量的类型是`Builtin.Int64`，



