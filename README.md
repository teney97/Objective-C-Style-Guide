# Objective-C-Style-Guide

## 目录

[if 语句](#if-语句)
* [1.将常量前置，以避免手误将 == 写成 = （Yoda Condition）](#将常量前置以避免手误将--写成--yoda-condition)
* [2.使用布尔表达式，而不是隐式地与 0 做对比](#使用布尔表达式而不是隐式地与-0-做对比)
* [3.复杂条件处理1 - 提高可调式性](#复杂条件处理1---提高可调式性)


## if 语句

### 将常量前置，以避免手误将 == 写成 = （Yoda Condition）

在 OC 中，赋值符 `=` 会有返回值，这导致我们有时候可能会手误将 `==` 写成 `=` 导致代码错误而不能尽早地发现问题。

```objc
- (void)method:(NSObject *)obj {
    if (obj = nil) { // warning!
        return;
    }
    //...
    // (obj == nil) is always true now
}
```

以上示例代码中的 `if (obj = nil)` 其实相当于如下代码，所以 `return` 语句将永远不会执行，且 `obj` 会被赋值为 `nil`。


```objc
temp = (obj = nil); // obj = nil, temp = nil
if (temp) {
    return;
}
```

你会收到 Xcode 的警告，但不会得到编译错误。

```
Using the result of an assignment as a condition without parentheses
1.Place parentheses around the assignment to silence this warning
2.Use '==' to turn this assignment into an equality comparison
```

可以看到，除了修改为正确的 `==` 运算符，加上一对括号就像 `if ((obj = nil))` 也是可以消除警告的。还有可能某些开发者根本不注意警告虽然这是不对的。甚至，你可能在 Podfile 文件中添加了 `inhibit_all_warnings!` 来忽略引入库的所有警告。

So，编译器的不严格给我们带来了安全隐患，那么我们该如何去规避这个问题呢？

将常量前置，比如：

```objc
if (nil == obj) { ... }
if (50 == height) { ... }
```

这样如果你手误将 `==` 写成 `=`，你将会得到编译错误，而不是警告。

```objc
- (void)method:(NSObject *)obj {
    if (nil = obj) { // Error: Expression is not assignable
        return;
    }
    //...
    // (obj == nil) is always true now
}
```

其实，这种写法有个专业名称，叫 `Yoda Condition`。   
> Yoda Condition 尤达条件式（也称为尤达标记法）是一种计算机编程中的编程风格，其中表达式的两个部分与条件语句中的典型顺序相反。尤达条件式将表达式的常量部分放在条件语句的左侧。这个风格的名称源于星际大战绝地大师尤达的角色，他以非标准语法讲英语（Instead of saying something like “you will try,” Yoda says “try, you will.”）。
>
>参考 wiki 百科：https://www.hk.wiiaa.top/baike-%E5%B0%A4%E9%81%94%E6%A2%9D%E4%BB%B6%E5%BC%8F

But，该写法不是万能的，有种情况下还得你自己注意拼写，那就是当被判断值与判断值都为变量的情况：

```objc
if (obj1 == obj2) { ... }
```

尽管该写法可以有效避免我们犯错，但它的可读性是有所降低的，就比如我们会说“如果天空是蓝色的”，而不会说“如果蓝色是天空”。如果你能铭记这个风险，时刻提醒自己要注意，那么使用 `if (obj == nil)` 更好。 

如果你使用 Swift，你就不必担心这个问题，不需要使用 `Yoda Condition`，因为在 Swift 中，赋值符 `=` 不再有返回值，这样就消除了手误将判等运算符 `==` 写成赋值符导致代码错误的缺陷。由此你也可以看出 Swift 比 Objective-C 更安全。

扩展：因为该差异，你可以在 OC 中同时对多个变量进行赋值，而 Swift 则不能。

```objc
// OC
width = height = 100; // height = 100, width = 100
// Swift
width = height = 100; // Error: Cannot assign value of type '()' to type 'typeOfWidth'
```

### 使用布尔表达式，而不是隐式地与 0 做对比

在 OC 中，if 语句中的条件可以是一个非 0 值，它将隐式地与 0 做对比，然后返回一个布尔值。你会经常看到以下这样的代码，你也可能习惯写这样的代码。

```objc
- (NSMutableArray *)mArray {
    if (!_mArray) {
        _mArray = [NSMutableArray arrayWithCapacity:10];
    }
    return _mArray;
}
```

该写法确实提高了代码简洁性，而且它可以避免手误将 `==` 写成 `=` 情况的发生，但它的可读性不如布尔表达式，所以我更推荐以下的写法：

```objc
- (NSMutableArray *)mArray {
    if (nil == _mArray) { // or (_mArray == nil)
        _mArray = [NSMutableArray arrayWithCapacity:10];
    }
    return _mArray;
}
```

对于布尔值，我更偏向于直接使用，而不是再将其转换为布尔表达式：

```objc
// Preferred:
if (isEnabled) { ... }
if (!isEnabled) { ... }
// Not Preferred:
if (isEnabled == YES) { ... }
if (isEnabled == NO) { ... }
```

但是，我的前辈更建议布尔值也转换为布尔表达式，以提高可读性，所以我觉得这个写法还是看个人习惯吧，或许以后我也会改变风格。

在 Swift 中，if 语句中的条件必须是一个布尔值 or 布尔表达式，这意味着像 `if score { ... }` 这样的代码将报错，而不会隐式地与 0 做对比。所以，养成这个编码习惯也能让我们更好地向 Swift 过渡。


### 复杂条件处理1 - 提高可调式性

当 if 语句中存在复杂条件时代码可调式性会降低，就如：

```objc
if ([obj1 isSuccess] || [obj2 isSuccess] || [obj3 isSuccess] && [obj3 isSupport]) {
    // ...
}
```

复杂条件会使你不知道是哪个子条件成立而使整个 if 条件成立，可调式性差。

在解决如何提高可调式性问题前，让我们先给 `[obj3 isSuccess] && [obj3 isSupport]` 加对 `()` 来明确运算符的优先级，以提高代码的可读性，虽然它并非必要，因为我们都知道 `&&` 优先级高于 `||`。

```objc
if ([obj1 isSuccess] || [obj2 isSuccess] || ([obj3 isSuccess] && [obj3 isSupport])) {
    // ...
}
```

那么，如何提高这段代码的可调式性呢？

我们需要考虑两种情况：

① 子条件计算不复杂

将每个子条件分别提取出来分配给一个 BOOL 变量，这样你就可以调试每个子条件的具体值。

```objc
BOOL isObj1Success = [obj1 isSuccess];
BOOL isObj2Success = [obj2 isSuccess];
BOOL isObj3Success = [obj3 isSuccess] && [obj3 isSupport];
if (isObj1Success || isObj2Success || isObj3Success) {
    // ...
}
```

② 子条件计算复杂，有一定耗时

如果子条件计算复杂，那以上方法就不适用了，因为这样会提前计算所有条件值。我们要利用 if 语句中使用复杂条件支持短路求值的天然优势来优化代码，
`[obj2 isSuccess]` 将会推迟到 `[obj1 isSuccess]` 判定为 `NO` 之后才会计算 ，如果 `[obj1 isSuccess]` 判定为 `YES`，整个 if 条件就成立，而不会再计算判定后面的子条件。这在子条件计算复杂的情况下，极大地提升了代码性能。

那么这种情况下，我们又该如何提高代码可调式性呢？

利用换行，我们就可以对每个子条件进行断点，如果 `[obj1 isSuccess]` 成立，那么之后的子条件的断点将不会执行到，以此类推。

```objc
if ([obj1 isSuccess] 
    || [obj2 isSuccess] 
    || ([obj3 isSuccess] && [obj3 isSupport])) {
    // ...
}
```
