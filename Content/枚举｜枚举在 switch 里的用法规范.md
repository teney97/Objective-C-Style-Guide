### 枚举在 switch 里的用法规范

在处理枚举类型的 switch 语句中，如果你想处理所有枚举类型，而不是给部分类型使用默认处理方式的话，那么就**不要实现 default 分支**。

这样的话，如果我们新加了一种枚举类型，那么编译器就会给我们警告，提示新加的枚举类型没在 switch 中处理。

```objectivec
typedef NS_ENUM(NSUInteger, EOCConnectionState) {
    EOCConnectionStateDisconnected,
    EOCConnectionStateConnecting,
    EOCConnectionStateConnected,
};

switch (connectionState) { // ⚠️ Enumeration value 'EOCConnectionStateConnected' not handled in switch
    case EOCConnectionStateDisconnected:
        // ...
        break;
    case EOCConnectionStateConnecting:
        // ...
        break;
}
```

而如果加了 default 分支，编译器就不会给警告了。这就不能确保 switch 语句能正确处理所有的枚举类型。

在 Swift 中，我们也可以根据实际使用场景这样做，如果 `switch` 语句没有穷举所有情况，那么代码将无法通过编译，强制穷举确保了枚举成员不会被意外遗漏。这点比 OC 更严格。

当然，如果你只是想在 switch 中处理部分枚举类型，加上 default 分支吧！

> 参考：《Effective Objective-C 2.0》第 5 条。
