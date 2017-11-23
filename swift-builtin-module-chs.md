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

![How Objective-C and Swift compilers works](/assets/fig_01.png)

如上图所示，Objective-C代码经过Clang处理后，生成**LLVM Intermediate Representation (IR)**，再经过LLVM处理，最后生成机器码。

可以把LLVM IR想象成某种高级的汇编语言，这种汇编语言不依赖于特定的架构，如i386或ARM等。
