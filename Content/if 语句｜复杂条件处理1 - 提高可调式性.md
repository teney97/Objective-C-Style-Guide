## 复杂条件处理1 - 提高可调式性

当 if 语句中存在复杂条件时代码可调式性会降低，就如：

```objc
if ([obj1 isSuccess] || [obj2 isSuccess] || [obj3 isSuccess] && [obj3 isSupport]) {
    // ...
}
```

复杂条件会使你不知道是哪个子条件成立而使整个 if 条件成立，可调式性差。

在解决如何提高可调式性问题前，让我们先给 `[obj3 isSuccess] && [obj3 isSupport]` 加对 `()` 来明确运算符的优先级，以提高代码的可读性（可读性 > 简洁性），虽然它并非必要，因为我们都知道 `&&` 优先级高于 `||`。

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
