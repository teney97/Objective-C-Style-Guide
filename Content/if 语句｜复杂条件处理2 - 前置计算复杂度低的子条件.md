## 复杂条件处理2 - 前置计算复杂度低的子条件

在上一点中我们提到，要利用 if 语句中使用复杂条件支持短路求值的天然优势来优化代码。为了代码最优化，我们需要将计算复杂度低的子条件前置。

以下是优化前的一段示例代码：

```objc
if ([obj isSupportWithComplexCalculation] 
    || [string isEqualToString:@"success"] 
    || aBoolValue) {
    // ...
}
```

以下是优化后的代码：

```objc
if (aBoolValue
    || [string isEqualToString:@"success"] 
    || [obj isSupportWithComplexCalculation] ) {
    // ...
}
