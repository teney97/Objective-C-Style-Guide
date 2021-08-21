## 使用布尔表达式，而不是隐式地与 0 做对比

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

但是，我的前辈更建议布尔值也转换为布尔表达式，以提高可读性，有时候确实如此。

比如：

```objc
// Preferred:
if (self.view.hidden == NO) { ... }
// Not Preferred:
if (!self.view.hidden) { ... }
```

因为 `hidden` 一词本身就带着相反的意思（`hidden == !show`），`!hidden` 对反取反，我觉得可读性不是很高，如果 `UIView` 提供的是 `show` 属性而不是 `hidden`，那我更倾向于：

```objc
// Preferred:
if (self.view.show) { ... }
if (!self.view.show) { ... }
// Not Preferred:
if (self.view.show == YES) { ... }
if (self.view.show == NO) { ... }
```

在 Swift 中，if 语句中的条件必须是一个布尔值 or 布尔表达式，这意味着像 `if score { ... }` 这样的代码将报错，而不会隐式地与 0 做对比。所以，养成这个编码习惯也能让我们更好地向 Swift 过渡。
