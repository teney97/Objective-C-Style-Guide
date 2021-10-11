## 混编｜为 Swift 改进 Objective-C API 声明（NS_REFINED_FOR_SWIFT）

使用宏 NS_REFINED_FOR_SWIFT 来改进 Objective-C API 声明。该宏在混编时主要参与适配器的工作，用途有：

* 你想在 Swift 中使用某个 Objective-C API 时，采用一些 Swift 的特有类型，比如元组（具体例子可以看 Example_Apple）
* 你想在 Swift 中使用某个 Objective-C API 时，重新排列、组合、重命名参数等等，以便该 API 与其它 Swift API 相匹配
* 当一组 Objective-C API 的关系为：其中有一个全能方法，其它方法均调用此方法，并为一些参数赋默认值，而且方法数量较多时，可以使用该宏将一些简单的不常用的方法隐式地标记为不可用，留下全能方法以及常用的方法。利用 Swift 可以为参数赋默认值的优势，来减少这组 Objective-C API 数量（具体例子可以看 Example_SDWebImage）
* 做一些兼容性的东西，比如 Swift 调用 Objective-C 的 API 时可能由于数据类型等不一致导致无法达到预期。例如，Objective-C 里的方法采用了 C 语言风格的多参数类型；或者 Objective-C 方法返回 NSNotFound，在 Swift 中期望返回 nil 等等（具体例子可以看 Example_Other）

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

默认情况下，该 API 导入到 Swift 中是下面这样的：

```swift
open func getRed(_ red: UnsafeMutablePointer<CGFloat>?, 
                 green: UnsafeMutablePointer<CGFloat>?, 
                 blue: UnsafeMutablePointer<CGFloat>?, 
                 alpha: UnsafeMutablePointer<CGFloat>?)
```

在调用时你需要传递四个 in-out 参数，显然这 API 在 Swift 中不够优雅。

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

怎么样？API 更简洁优雅了，使用起来也更方便，同时用到了 Swift 的特有类型 —— 元组。

接下来我们来看看，如何通过使用 NS_REFINED_FOR_SWIFT 做这样一个 API 适配。

#### 1. 将宏 NS_REFINED_FOR_SWIFT 作为后缀添加到 Objective-C API 中

首先，将宏 NS_REFINED_FOR_SWIFT 作为后缀添加到 Objective-C API 中。该 Objective-C API 在导入到 Swift 中时，会以双下划线 (`__`) 开头重命名，且在 Swift 中调用时不会有代码提示，相当于隐藏了 API。这样可以一定程度上防止你意外地直接使用该 Objective-C API，而没有使用适配后的 Swift API。 

> 这里是可以一定程度上防止而不是绝对，因为如果开发者知道该规则的话，仍然可以以 (`__`) 开头拼接 Objective-C API 名称调用。但既然使用了 NS_REFINED_FOR_SWIFT 做 API 适配，那就遵守规范吧，不要这样使用！

```objectivec
@interface Color : NSObject
 
- (void)getRed:(nullable CGFloat *)red
         green:(nullable CGFloat *)green
          blue:(nullable CGFloat *)blue
         alpha:(nullable CGFloat *)alpha NS_REFINED_FOR_SWIFT;
 
@end
```

#### 2. 在 Swift 中添加适配器 API

在 Swift 中添加一个新的 API，来对 Objective-C API 进行适配改进。这里就是实现只读计算属性 rgba，在实现中调用以 (`__`) 开头重命名的 Objective-C API。

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

现在调用是不是优雅多了？

```swift
let color = Color()
var r = color.rgba.red
var g = color.rgba.green
var b = color.rgba.blue
var a = color.rgba.alpha
```

### Example_SDWebImage

接下来让我们看看 SDWebImage 是怎么使用 NS_REFINED_FOR_SWIFT 的。

UIImageView (WebCache) 分类中扩展的方法非常多吧？总共有 9 个，这都是因为 Objective-C 中不能给方法参数赋默认值的缘故 😭。那么在 Objective-C API 导入到 Swift 时，如何巧妙地利用上 Swift 可以为方法参数赋默认值的优点呢？答案就是使用 NS_REFINED_FOR_SWIFT 宏，SDWebImage 为其中 4 个方法添加上了宏 NS_REFINED_FOR_SWIFT。

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

这样在 Swift 中调用的时候只会提示另外 5 个 API。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1629873847844-91a338b5-9100-40b4-8e65-642b07e092a8.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

你还是可以调用 `sd_setImage(with url: URL?, placeholderImage placeholder: UIImage?)`：

```swift
let imageView = UIImageView()
imageView.sd_setImage(with: nil, placeholderImage: nil)
```

但这时候它不是调用 Objective-C 的 `- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder` 方法，而是调用 `- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder options:(SDWebImageOptions)options completed:(nullable SDExternalCompletionBlock)completedBlock;` 方法，并为 `options`、`completed` 参数赋默认值。

> 如果你想调用 Objective-C 的 `- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder` 方法，那就以双下划线 (`__`) 开头调用，但是不建议这样使用，请遵守规范！

```swift
// Objective-C API
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
// In Swift, this API is imported like this:
open func sd_setImage(with url: URL?, placeholderImage placeholder: UIImage?, options: SDWebImageOptions = [], completed completedBlock: SDExternalCompletionBlock? = nil)
```

如何验证呢？

很简单，我们可以使用 `NS_SWIFT_UNAVAILABLE` 宏将 `- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder options:(SDWebImageOptions)options completed:(nullable SDExternalCompletionBlock)completedBlock;` 方法标记为在 Swift 中不可用，这时候就编译错误了：

```swift
// Objective-C API
- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                 completed:(nullable SDExternalCompletionBlock)completedBlock NS_SWIFT_UNAVAILABLE("Unavailable");
// Use in Swift
let imageView = UIImageView()
imageView.sd_setImage(with: nil, placeholderImage: nil) // Error: 'sd_setImage(with:placeholderImage:options:completed:)' is unavailable in Swift: Unavailable
```

我们再利用 `NS_SWIFT_UNAVAILABLE` 宏来看看这 4 个用 `NS_REFINED_FOR_SWIFT` 标记的 API 最终都是调用哪个方法：

```swift
let imageView = UIImageView()
imageView.sd_setImage(with: nil)
// Error: 'sd_setImage(with:completed:)' is unavailable in Swift: Unavailable
imageView.sd_setImage(with: nil, placeholderImage: nil)
// Error: 'sd_setImage(with:placeholderImage:options:completed:)' is unavailable in Swift: Unavailable
imageView.sd_setImage(with: nil, placeholderImage: nil, options: .retryFailed)
// Error: 'sd_setImage(with:placeholderImage:options:completed:)' is unavailable in Swift: Unavailable
imageView.sd_setImage(with: nil, placeholderImage: nil, completed: nil)
// Error: 'sd_setImage(with:placeholderImage:options:completed:)' is unavailable in Swift: Unavailable
```

现在，你是不是对 `NS_REFINED_FOR_SWIFT` 宏的用法掌握得更多了呢？

### Example_Other

NS_REFINED_FOR_SWIFT 宏也用于做一些兼容性的东西，比如 Swift 调用 Objective-C 的 API 时可能由于数据类型等不一致导致无法达到预期。例如，Objective-C 里的方法采用了 C 语言风格的多参数类型；或者 Objective-C 方法返回 NSNotFound，在 Swift 中期望返回 nil 等等。

```objectivec
@interface MyClass : NSObject

- (NSUInteger)indexOfString:(NSString *)aString NS_REFINED_FOR_SWIFT; 
// 这里使用 NS_REFINED_FOR_SWIFT，该方法名在 Swift 中会被重新命名为 __index(of: aString)

@end
```

```swift
extension MyClass {
    // 扩展 MyClass 并重新定义 indexOfString 函数
    func indexOfString(aString: String!) -> Int? { 
        let index = Int(__index(of: aString)) // 这里的 __index 是 OC 中的 indexOfString 方法
        if (index == NSNotFound) {
            return nil    
        }
        return index
    }
}
```

### NS_REFINED_FOR_SWIFT 宏对 Objective-C API 的重命名规则

添加了 NS_REFINED_FOR_SWIFT 的 Objective-C API 在导入到 Swift 时，具体的 API 重命名规则如下：

* 如果是初始化方法，则在其第一个参数标签前面加 (`__`) 

  ```swift
  // Objective-C API
  - (instancetype)initWithColor:(UIColor *)color NS_REFINED_FOR_SWIFT;
  // Use in Swift
  let color = Color(__color: .red)
  ```

* 如果是属性，在属性名前面加 (`__`) 

  ```swift
  // Objective-C API
  @property (nonatomic, copy) NSString *name NS_REFINED_FOR_SWIFT;
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

> 注意：NS_REFINED_FOR_SWIFT 和 NS_SWIFT_NAME 一起用的话，NS_REFINED_FOR_SWIFT 不生效，而是以 NS_SWIFT_NAME 指定的名称重命名 Objective-C API。

### 参考

* [Apple｜Improving Objective-C API Declarations for Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/improving_objective-c_api_declarations_for_swift)
* [Jacob Bandes-Storch｜Help Yourself to Some Swift](https://bandes-stor.ch/blog/2015/11/28/help-yourself-to-some-swift/)
* [Apple｜WWDC20 10680 - Refine Objective-C frameworks for Swift](https://developer.apple.com/videos/play/wwdc2020/10680/)
* [小专栏｜WWDC20 10680 - 让 Objective-C 框架与 Swift 友好共存的秘籍](https://xiaozhuanlan.com/topic/1980624753)

