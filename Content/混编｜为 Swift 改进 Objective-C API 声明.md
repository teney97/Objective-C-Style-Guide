## 混编｜为 Swift 改进 Objective-C API 声明

使用宏 NS_REFINED_FOR_SWIFT 来改进 Objective-C API 声明。该宏在混编时主要参与适配器的工作，用途有：

* 你想在 Swift 中使用某个 Objective-C API 时，采用一些 Swift 的特有类型，比如元组
* 你想在 Swift 中使用某个 Objective-C API 时，重新排列、组合、重命名参数等等，以便该 API 与其它 Swift API
* 做一些兼容性的东西，比如 Swift 调用 Objective-C 的 API 时可能由于数据类型等不一致导致无法达到预期。例如，Objective-C 里的方法采用了 C 语言风格的多参数类型；或者 Objective-C 方法返回 NSNotFound，在 Swift 中期望返回 nil 等等
* 如果你只是简单的在 Swift 中定义一个新函数，函数里调用这个 Objective-C API，相当于是为 Swift 重命名 Objective-C API，这用 NS_SWIFT_NAME 就行，没必要使用 NS_REFINED_FOR_SWIFT

### Example 1

这个是 [Apple｜Improving Objective-C API Declarations for Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/improving_objective-c_api_declarations_for_swift) 的例子。

以下是在 Objective-C 中声明的 API

```objectivec
@interface Color : NSObject
 
- (void)getRed:(nullable CGFloat *)red
         green:(nullable CGFloat *)green
          blue:(nullable CGFloat *)blue
         alpha:(nullable CGFloat *)alpha;
 
@end
```

默认情况下，在 Swift 中调用是如下这样的，你需要传递四个 in-out 参数，显然这 API 在 Swift 中不够优雅。

```swift
color.getRed(red: UnsafeMutablePointer<CGFloat>?, 
             green: UnsafeMutablePointer<CGFloat>?, 
             blue: UnsafeMutablePointer<CGFloat>?, 
             alpha: UnsafeMutablePointer<CGFloat>?)
```

如果你在 Swift 中设计这样一个 API，你会如何设计呢？

可以设计一个计算属性，其类型是一个包含 rgba 四个元素的元组。

```swift
var rgba: (red: CGFloat, green: CGFloat, blue: CGFloat, alpha: CGFloat)
```

怎么样？API 更简洁优雅了，使用起来也更方便，同时用到了 Swift 的特有类型 —— 元组。

接下来我们来看看，如何通过使用 NS_REFINED_FOR_SWIFT 做一个 API 适配。

#### 1. 将宏 NS_REFINED_FOR_SWIFT 作为后缀添加到 Objective-C API 中

首先，将宏 NS_REFINED_FOR_SWIFT 作为后缀添加到 Objective-C API 中。该 Objective-C API 在导入到 Swift 中时，会以双下划线 (`__`) 开头重命名，且在 Swift 中调用时不会有代码提示。这样可以一定程度上防止你意外地直接使用该 Objective-C API，而没有使用适配后的 Swift API。 

> 这里是可以一定程度上防止而不是绝对，因为如果开发者知道该规则的话，仍然可以以 (`__`) 开头拼接 Objective-C API 名称调用。但既然使用了 NS_REFINED_FOR_SWIFT 做 API 适配，那就遵守规范吧！

```objectivec
@interface Color : NSObject
 
- (void)getRed:(nullable CGFloat *)red
         green:(nullable CGFloat *)green
          blue:(nullable CGFloat *)blue
         alpha:(nullable CGFloat *)alpha NS_REFINED_FOR_SWIFT;
 
@end
```

添加了 NS_REFINED_FOR_SWIFT 的 Objective-C API 在导入到 Swift 时，具体的 API 重命名规则如下：

* 如果是初始化方法，则在其第一个参数标签前加 (`__`) 

  ```swift
  // Objective-C API
  - (instancetype)initWithColor:(UIColor *)color NS_REFINED_FOR_SWIFT;
  // Use in Swift
  let color = Color(__color: .red)
  ```

* 如果是 getter 或 setter 方法

  ```swift
  // Objective-C API
  @property (nonatomic, assign) NSString *name NS_REFINED_FOR_SWIFT;
  // Use in Swift
  object.__name = "zhangsan"
  ```

* 其它方法，在方法名前面加 (`__`) 

  ```swift
  // Objective-C API
  + (void)method NS_REFINED_FOR_SWIFT;
  // Use in Swift
  Color.__method()
  ```

#### 2. 在 Swift 中添加适配器 API

在 Swift 中添加一个新的 API，来对 Objective-C API 进行适配改进。这里就是实现计算属性 rgba，在实现中调用以 (`__`) 开头重命名的 Objective-C API。

```swift
extension Color {
    var rgba: (red: CGFloat, green: CGFloat, blue: CGFloat, alpha: CGFloat) {
        var r: CGFloat = 0.0
        var g: CGFloat = 0.0
        var b: CGFloat = 0.0
        var a: CGFloat = 0.0
        __getRed(red: &r, green: &g, blue: &b, alpha: &a)
        return (red: r, green: g, blue: b, alpha: a)
    }
}
```

### 参考

* [Apple｜Improving Objective-C API Declarations for Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/improving_objective-c_api_declarations_for_swift)

