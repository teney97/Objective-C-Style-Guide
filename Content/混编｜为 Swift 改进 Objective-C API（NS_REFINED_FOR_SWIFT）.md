## 混编｜为 Swift 改进 Objective-C API（NS_REFINED_FOR_SWIFT）

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1634031406142-5b7965cc-bbab-4559-978d-ce3e05b52abf.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

关键词：NS_REFINED_FOR_SWIFT

### 前言

宏 `NS_REFINED_FOR_SWIFT` 于 [Xcode 7](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW326) 引入，它可用于在 Swift 中隐藏 Objective-C API，以便在 Swift 中提供相同 API 的更好版本，同时仍然可以使用原始 Objective-C 实现。具体的应用场景有：

* 你想在 Swift 中使用某个 Objective-C API 时，使用不同的方法声明，但要使用类似的底层实现
* 你想在 Swift 中使用某个 Objective-C API 时，采用一些 Swift 的特有类型，比如元组（具体例子可以看 Example_Apple）
* 你想在 Swift 中使用某个 Objective-C API 时，重新排列、组合、重命名参数等等，以使该 API 与其它 Swift API 更匹配
* 利用 Swift 支持默认参数值的优势，来减少导入到 Swift 中的一组 Objective-C API 数量（具体例子可以看 Example_SDWebImage）
* 解决 Swift 调用 Objective-C 的 API 时可能由于数据类型等不一致导致无法达到预期的问题。例如，Objective-C 里的方法采用了 C 风格的多参数类型；或者 Objective-C 方法返回 NSNotFound，在 Swift 中期望返回 nil 等等（具体例子可以看 Example_Other）

### Example_Apple

这个是 [Apple｜Improving Objective-C API Declarations for Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/improving_objective-c_api_declarations_for_swift) 的例子。

以下是在 Objective-C 中声明的 API：

```objectivec
@interface Color : NSObject
 
- (void)getRed:(nullable CGFloat *)red
         green:(nullable CGFloat *)green
          blue:(nullable CGFloat *)blue
         alpha:(nullable CGFloat *)alpha;
 
@end
```

默认情况下，生成的 Swift API 是下面这样的：

```swift
open func getRed(_ red: UnsafeMutablePointer<CGFloat>?, 
                 green: UnsafeMutablePointer<CGFloat>?, 
                 blue: UnsafeMutablePointer<CGFloat>?, 
                 alpha: UnsafeMutablePointer<CGFloat>?)
```

在 Swift 中调用该方法需要传递四个 in-out 参数，易用性较低。

```swift
let color = Color()
var r: CGFloat = 0.0
var g: CGFloat = 0.0
var b: CGFloat = 0.0
var a: CGFloat = 0.0
color.getRed(&r, green: &g, blue: &b, alpha: &a)
```

如果你在 Swift 中设计这样一个 API，你会如何设计呢？

可以设计一个只读计算属性，其类型是一个包含 rgba 四个元素的元组。

```swift
var rgba: (red: CGFloat, green: CGFloat, blue: CGFloat, alpha: CGFloat)
```

怎么样？API 更简洁更易用了，同时用到了 Swift 的特有类型 —— 元组。

接下来我们来看看，如何通过使用 `NS_REFINED_FOR_SWIFT` 做这样一个 API 适配。

#### 1. 将宏 NS_REFINED_FOR_SWIFT 添加到 Objective-C API 中

首先，将 `NS_REFINED_FOR_SWIFT` 作为后缀添加到 Objective-C API 中，生成的 Swift API 将会以双下划线 `__` 开头重命名，且在 Swift 中调用时不会有代码补全提示，相当于隐藏了 API。这样可以一定程度上防止调用者意外地直接使用该 Objective-C API，而没有使用适配后的 Swift API。 

> 这里是可以一定程度上防止而不是绝对，因为如果调用者了解该规则的话，仍然可以以 `__` 开头拼接 Objective-C API 名称进行调用。但既然使用了 `NS_REFINED_FOR_SWIFT` 做 API 适配，就表明该方法不应该原样使用，遵守规范吧！

```objectivec
@interface Color : NSObject
 
- (void)getRed:(nullable CGFloat *)red
         green:(nullable CGFloat *)green
          blue:(nullable CGFloat *)blue
         alpha:(nullable CGFloat *)alpha NS_REFINED_FOR_SWIFT;
 
@end
```

#### 2. 在 Swift 中添加适配 API

在 Swift 中添加一个新的 API，来对 Objective-C API 进行适配改进。这里就是实现只读计算属性 rgba，在实现中调用以 `__` 开头重命名的 Objective-C API。

```swift
extension Color {
    var rgba: (red: CGFloat, green: CGFloat, blue: CGFloat, alpha: CGFloat) {
        var r: CGFloat = 0.0
        var g: CGFloat = 0.0
        var b: CGFloat = 0.0
        var a: CGFloat = 0.0
        __getRed(&r, green: &g, blue: &b, alpha: &a)
        return (red: r, green: g, blue: b, alpha: a)
    }
}
```

现在调用方式是不是好极了！

```swift
let color = Color()
var r = color.rgba.red
var g = color.rgba.green
var b = color.rgba.blue
var a = color.rgba.alpha
```

### Example_SDWebImage

接下来让我们看看 SDWebImage 是怎么使用 `NS_REFINED_FOR_SWIFT` 的。

Objective-C 方法不支持默认参数值，通常是提供一个多参数的全能方法，然后提供几个少参数的便利方法来调用全能方法，并为一些参数赋默认值。这种方式组合不灵活且扩展不方便，而且在迭代过程中这一组 API 数量可能会越来越多。像 UIImageView (WebCache) 分类中扩展的方法就多达 9 个。

那么，在 Objective-C API 导入到 Swift 时，如何利用上 Swift 可以为方法参数赋默认值的优点呢？可以使用 `NS_REFINED_FOR_SWIFT` 宏，SDWebImage 为其中 4 个方法添加上了该宏。

```objectivec
// UIImageView+WebCache.h
- (void)sd_setImageWithURL:(nullable NSURL *)url NS_REFINED_FOR_SWIFT;
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder NS_REFINED_FOR_SWIFT;
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options NS_REFINED_FOR_SWIFT;
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                 completed:(nullable SDExternalCompletionBlock)completedBlock NS_REFINED_FOR_SWIFT;
```

在 Swift 中调用的时候，代码补全只会提示剩下的 5 个 API。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1629873847844-91a338b5-9100-40b4-8e65-642b07e092a8.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

你还是可以调用 **_sd_setImage(with url: URL?, placeholderImage placeholder: UIImage?)_**：

```swift
imageView.sd_setImage(with: nil, placeholderImage: nil)
```

但这时候它不是调用 Objective-C 的 **_- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder;_** 方法，而是调用 **_- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder options:(SDWebImageOptions)options completed:(nullable SDExternalCompletionBlock)completedBlock;_** 方法，并为 options、completed 参数赋默认值。

> 如果你想调用 Objective-C 的 **_- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder_** 方法，那就以双下划线 `__` 开头调用，但是不建议更没必要这样使用，请遵守规范！

```swift
// Objective-C API
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                 completed:(nullable SDExternalCompletionBlock)completedBlock;

// In Swift, this API is imported like this:
open func sd_setImage(with url: URL?, placeholderImage placeholder: UIImage?, options: SDWebImageOptions = [], completed completedBlock: SDExternalCompletionBlock? = nil)
```

我们来看看 Generated Swift Interface：

```swift
extension UIImageView {
    open func sd_setImage(with url: URL?, placeholderImage placeholder: UIImage?, options: SDWebImageOptions = [], context: [SDWebImageContextOption : Any]?)
    open func sd_setImage(with url: URL?, completed completedBlock: SDExternalCompletionBlock? = nil)
    open func sd_setImage(with url: URL?, placeholderImage placeholder: UIImage?, options: SDWebImageOptions = [], completed completedBlock: SDExternalCompletionBlock? = nil)
    open func sd_setImage(with url: URL?, placeholderImage placeholder: UIImage?, options: SDWebImageOptions = [], progress progressBlock: SDImageLoaderProgressBlock?, completed completedBlock: SDExternalCompletionBlock? = nil)
    open func sd_setImage(with url: URL?, placeholderImage placeholder: UIImage?, options: SDWebImageOptions = [], context: [SDWebImageContextOption : Any]?, progress progressBlock: SDImageLoaderProgressBlock?, completed completedBlock: SDExternalCompletionBlock? = nil)
}
```

由此，我们可以推断出，Objective-C 方法在导入到 Swift 中时，哪些参数支持默认值：

* 可选枚举
* 作为最后一个参数的 block

### Example_Other

`NS_REFINED_FOR_SWIFT` 宏也可以用于解决 Swift 调用 Objective-C 的 API 时可能由于数据类型等不一致导致无法达到预期的问题。例如，Objective-C 里的方法采用了 C 风格的多参数类型；或者 Objective-C 方法返回 NSNotFound，在 Swift 中期望返回 nil 等等。

举个具体的例子：

```objectivec
// Declare in Objective-C
@interface MyClass : NSObject
- (NSInteger)indexOfString:(NSString *)aString; 
@end
  
// Generated Swift Interface
open func index(of aString: String) -> Int
```

这个 Objective-C API 在 Swift 中使用没有问题，不足的地方在于它将返回一个有效的 NSInteger 值或者 NSNotFound，而在 Swift 中我们更期望的是它返回一个 Int 或者 nil（也就是返回一个 Int?），这样就可以使用可选绑定了。

下面我们来看看如何为该 Objective-C API 做适配。

首先，该 API 添加上宏 `NS_REFINED_FOR_SWIFT`，它在 Swift 中会就被重命名为 `__index(of: aString)`，且不会在 Generated Swift Interface 中显示。

```objectivec
- (NSUInteger)indexOfString:(NSString *)aString NS_REFINED_FOR_SWIFT; 
```

其次，在 Swift 中 extension MyClass 并重新定义 indexOfString 方法，调用 `__index(of: aString)` 即调用 Objective-C 中的 indexOfString 方法，判断返回值如果为 NSNotFound，就返回 nil，这样就完成了 API 的适配。

```swift
extension MyClass {
    func index(of aString: String) -> Int? { 
        let index = Int(__index(of: aString)) 
        if (index == NSNotFound) {
            return nil    
        }
        return index
    }
}
```

### NS_REFINED_FOR_SWIFT 宏对 Objective-C API 的重命名规则

`NS_REFINED_FOR_SWIFT` 可用于初始化方法、属性、其它方法。添加了 `NS_REFINED_FOR_SWIFT` 的 Objective-C API 在导入到 Swift 时，具体的 API 重命名规则如下：

* 如果是初始化方法，则在其第一个参数标签前面加 "__"

```swift
// Objective-C API
- (instancetype)initWithColor:(UIColor *)color NS_REFINED_FOR_SWIFT;
// Use it in Swift
let color = Color(__color: .red)
```

* 其它方法，在方法名前面加 "__"


```swift
// Objective-C API
+ (void)method NS_REFINED_FOR_SWIFT;
// Use it in Swift
Color.__method()
```

* 下标方法将被视为任何其它方法，在方法名前面加 "__"，而不是作为 Swift 下标导入
* 其他声明将在其名称前加上 "__"，例如属性：

```swift
// Objective-C API
@property (nonatomic, copy) NSString *name NS_REFINED_FOR_SWIFT;
// Use it in Swift
object.__name = "zhangsan"
```

> 注意：`NS_REFINED_FOR_SWIFT` 和 `NS_SWIFT_NAME` 一起用的话，`NS_REFINED_FOR_SWIFT` 不生效，而是以 `NS_SWIFT_NAME` 指定的名称重命名 Objective-C API。

### 小结

本文通过几个例子讲解了宏 `NS_REFINED_FOR_SWIFT` 的应用场景，让 Objective-C API 在 Swift 中更好地呈现，相信它在你混编的过程中一定会有用武之地的！

### 参考

* [Apple｜Xcode Release Notes - Xcode 7](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW326)
* [Apple｜Improving Objective-C API Declarations for Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/improving_objective-c_api_declarations_for_swift)
* [Apple｜WWDC20 10680 - Refine Objective-C frameworks for Swift](https://developer.apple.com/videos/play/wwdc2020/10680/)
* [Jacob Bandes-Storch｜Help Yourself to Some Swift](https://bandes-stor.ch/blog/2015/11/28/help-yourself-to-some-swift/)
* [Medium｜Adapting Objective-C APIs to Swift With NS_REFINED_FOR_SWIFT](https://betterprogramming.pub/adapting-objective-c-apis-to-swift-with-ns-refined-for-swift-fc66ca88ea51)
