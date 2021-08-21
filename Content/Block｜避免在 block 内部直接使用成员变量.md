## 避免在 block 内部直接使用成员变量

当我们在 block 内部直接使用 _variable 成员变量时，编译器会给我们警告：`Block implicitly retains self; explicitly mention 'self' to indicate this is intended behavior`。

原因是 block 中直接使用 _variable 会导致 block 隐式的强引用 self。Xcode 认为这可能会隐式的导致循环引用，从而给开发者带来困扰，而且如果不仔细看的话真的不太好排查，笔者之前就因为这个循环引用找了半天，还拉上了我导师一起查找原因。所以警告我们要显式的在 block 中使用 self，以达到 block 显式 retain 住 self 的目的。改用 `self->_variable` 或者 `self.variable`。

**Preferred:**

```objectivec
// Preferred:
@weakify(self);
self.block = ^{
    @strongify(self);
    NSLog(@"%@", self->_variable);
};
```

**Not Preferred:**

```objectivec
// Not Preferred:
@weakify(self);
self.block = ^{
    @strongify(self);
    NSLog(@"%@", _variable); // Block implicitly retains 'self'; explicitly mention 'self' to indicate this is intended behavior
};
```

你可能会觉得这种困扰没什么，如果你使用 `@weakify` 和 `@strongify` 那确实不会造成循环引用，因为 `@strongify` 声明的变量名就是 self。那如果你使用 `__weak typeof(self) weak_self = self;` 和 `__strong typeof(weak_self) strong_self = weak_self` 呢？
