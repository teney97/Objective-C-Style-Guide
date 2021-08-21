## 使用 Yoda Condition，将常量前置，以避免手误将 == 写成 =

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
> 参考：[wiki: 尤达条件式](https://www.hk.wiiaa.top/baike-%E5%B0%A4%E9%81%94%E6%A2%9D%E4%BB%B6%E5%BC%8F)

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
