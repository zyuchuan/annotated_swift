# [Swift's mysterious Builtin module](http://ankit.im/swift/2016/01/12/swift-mysterious-builtin-module/)
 
You might have noticed something like this if you `cmd + click Int` type in playground:
 
```
public struct Int : SignedIntegerType, Comparable, Equatable {

    public var value: Builtin.Int64
 
    ...
 
```
 or if you've been looking at source code of Swift's [stdlib](https://github.com/apple/swift/tree/master/stdlib/public/core) then you probably noticed a lot of `Builtin.*` functions, e.g.:
 
 * Builtin.Int1
 * Builtin.RawPointer
 * BuiBuiltin.NativeObject
 * Builtin.allocRaw
 * Builtin.alignof
 
 So what exactly is this mystical `Builtin`?
 
 ## Clang, Swift Compiler, SIL, IR, LLVM
 
To understand the true purpose and need for `Builtin` lets take a quick abstract overview on how Objective-C and Swift compiler works.
 
 ![](/assets/fig_01.png)
 
(Lots more happen in between but this is good enough for this post)

Objective-C code goes into clang which produces something called LLVM Intermediate Representation (IR) which is then fed into LLVM and the binary comes out.

LLVM IR is kind of a high-level assembly language independent of Arch like i368, ARM, etc. To create a compiler for a new language using LLVM, one just needs to implement a frontend which can compile code into LLVM IR and then its upto LLVM to generate correct assembly and binary for any platform that it supports.

![](/assets/fig_02.png)
 
Swift first creates SIL(Swift Intermediate Representation) which is then onverted into LLVM IR and then compiled by LLVM compiler.
 
As you can guess SIL is swifty wrapper over LLVM IR which was created for many reasons for eg: making sure variables are initialized before use, detecting unreachable code, optimization of code before sending it to LLVM etc. You can watch [this](https://www.youtube.com/watch?v=Ntj8ab-5cvE) talk to find out more why SIL exists and what it does.

The main takeaway here is the LLVM IR. For simple swift program like this:

```
let a = 5
let b = 6
let c = a + b
```

Looks like this in LLVM IR(can be generated using `swiftc -emit-ir addswfit.swfit`):

```
...
  store i64 5, i64* getelementptr inbounds (%Si* @_Tv8addswift1aSi, i32 0, i32 0), align 8
  //^ store 5 in a
  store i64 6, i64* getelementptr inbounds (%Si* @_Tv8addswift1bSi, i32 0, i32 0), align 8
  //^ store 6 in b
  %5 = load i64* getelementptr inbounds (%Si* @_Tv8addswift1aSi, i32 0, i32 0), align 8
  //^ load a to virtual register %5
  %6 = load i64* getelementptr inbounds (%Si* @_Tv8addswift1bSi, i32 0, i32 0), align 8
  //^ load b to virtual register %6
  %7 = call { i64, i1 } @llvm.sadd.with.overflow.i64(i64 %5, i64 %6)
  //^ call llvm's signed addition with overflow on %5 and %6 (returns two values: sum and a flag if overflowed)
  %8 = extractvalue { i64, i1 } %7, 0 
  //^ extract first value to %8
  %9 = extractvalue { i64, i1 } %7, 1
  //^ extract second value to %9
  br i1 %9, label %11, label %10
  //^ if overflowed jump to trap otherwise jump to label 10

; <label>:10                                      ; preds = %once_done
  store i64 %8, i64* getelementptr inbounds (%Si* @_Tv8addswift1cSi, i32 0, i32 0), align 8
  //^ store result in c
  ret i32 0
...
```

Find my comments about the generated IR after `//^` cirresponding to the line above it.

Even if the above code looks like garbage to you just note these two things:

* There is a data type in LLVM called `i64` which is 64 bit integer
* There is a method in LLVM IR called `llvm.sadd.with.overflow.i64` which adds two `i64` and return two things: sum and one big flag if addition failed.

## Explain Builtin already

Okay back to Swift, so we know that Swift `Int` is actually a Swift struct and `+` us actually a global function overloaded with `lhs` and `rhs` as `Int`. They are not part of the language in the sense that its understood by language directly for eg `struct`, `class`, `if`, `guard` etc are part of language. 

`Int` and `+` are part of Swift's stdlib which means they are not native constructs, which in turn means overheads == swift is SLOW? nope.

This is where `Builtin` comes in. `Builtin` exposes LLVM IR's types and methods directly to the stdlib so there is no overhead of looking up things at runtime and still make `Int` behave as an struct to do things like

```
extension Int {
    func times(otherInt: Int) -> Int { 
        return self * otherInt
    }
};

5.times(6)
```

Swift `struct Int` contains one single stored property called `value` which is type `Builtin.Int64` so we can use `unsafeBitCast` on it to convert back and forth but `stdlib` also provides an overloaded `init` to get Swift `Int` from `Builtin.Int64`.

Similarly `UnsafePointer` and related classes are wrapper over `Builtin`'s method which are related direct memory access. for eg: `alloc` method is defined as: 

```
public static func alloc(num: Int) -> UnsafeMutablePointer {
    let size = strideof(Memory.self) * num
    return UnsafeMutablePointer(
        Builtin.allocRaw(size._builtinWordValue, Builtin.alignof(Memory.self)))
```

Now we know why using Swift `Int` won't cause performance issues but what about `+` ioerator. It is still a function. It is defined as:

```
@_transparent
public func + (lhs: Int, rhs: Int) -> Int {
    let (result, error) = Builtin.sadd_with_overflow_Int64(
        lhs._value, rhs._value, true._value)
        
  // return overflowChecked((Int(result), Bool(error)))
  Builtin.condfail(error)
  return Int(result)
}
```

* `@_transparent` means this function should be inlined when called.
* `Builtin.sadd_with_overflow_Int64` corresponds to `llvm.sadd.with.overflow.i64` we saw earlier in LLVM IR which will return the tuple of `Builtin.Ing64:result` and `Builtin.Int1:error`.
* The result is converted back to Swift `Int` using `Int(result)` and returned.

So if these things are going to be inlined, it means this will produce good LLVM IR code which will produce good fast binary :)

## Can I play around with Builtin?

Builtin in swift is only available to stdlib and not normal Swift programs because of obvious reasons. But we can play with Builtin using `-parse-stdlib` flag of `swiftc`.

Example:

```
import Swift // import swift stdlib

let result = Builtin.sadd_with_overflow_Int64(5.value, 6.value, true._getBuiltinLogicValue())
print(Int(result.0))

let result2 = Builtin.sadd_with_overflow_Int64(unsafeBitCast(5, Builtin.Int64), unsafeBitCast(6, Builtin.Int64), true._getBuiltinLogicValue())
print(unsafeBitCast(result2.0, Int.self))
```

```
swiftc -parse-stdlib add.swift && ./add
```