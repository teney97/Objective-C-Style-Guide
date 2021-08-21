## 使用卫语句提前退出

>什么是卫语句？
>
>卫（guard）是布尔表达式，其结果必须为真，程序才能执行下去。卫语句用于代码的提前退出，可用来减少代码嵌套层级使代码更扁平，比如把 `if guard { ... }` 替代为：`if not guard: return; ...`
>
>参考自：[wiki: 卫语句](https://www.hk.wiiaa.top/wiki/%E5%8D%AB%E8%AF%AD%E5%8F%A5)


对于不满足条件的情况做提前退出处理，这在 if-else 语句层级嵌套多的情况下非常有用，可以减少嵌套，降低代码复杂性，提高可读性。将方法实现的重要部分放在外层（条件判断都成立的下方），而不是嵌套在分支内部，使代码更直观。

以下是优化前的一段示例代码：

```objc
if (string != nil && ![string isEqualToString:@""]) {
    // do something 1
    if (obj != nil) {
        // do something 2
        if ([obj isSuccess]) {
            // do something important
        }
    } else {
        // do something 3
    }
}
```

以下是优化后的代码：

```objc
if (string == nil || [string isEqualToString:@""]) {
    return
}

// do something 1

if (obj == nil) {
    // do something 3
    return;
}

// do something 2

if (![obj isSuccess]) {
    return;
}

// do something important

```

在 Swift 中，我们有了更好的卫语句实现方式 —— 使用 guard 语句。

如果条件为真，则继续往下执行代码，否则执行 else 中的代码。else 代码中必须转移控制以退出 guard 语句，比如 return、break、continue、throw，或者调用一个不返回的方法或函数，例如 fatalError()。

```Swift
guard string != nil && string!.isEmpty else {
    return
}

// do something 1

guard obj != nil else {
    // do something 3
    return
}

// do something 2

guard obj!.isSuccess() else {
    return
}

// do something important
```

你可能会问，使用 if 来实现卫语句可读性也不差，为什么 Swift 还要引入 guard 语句呢？

像 if 语句一样，guard 的执行取决于一个布尔值或布尔表达式。我们可以使用 guard 语句来要求条件必须为真时，才执行 guard 语句后的代码。不同于 if 语句，一个 guard 语句总是有一个 else 从句，如果条件不为真则执行 else 从句中的代码，且强制在 else 从句中转移控制以退出 guard 语句。而 if 语句则没有这么严格的要求。可以说 guard 语句是真正意义上的卫语句，而 if 语句只是我们将它写成了卫语句的形式。

另外，之前在网上有看到说，老外觉得在 if 语句条件中取反条件不好理解，而 guard 语句可读性更高。
