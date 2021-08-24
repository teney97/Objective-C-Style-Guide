## 混编｜为 Swift 重命名 Objective-C API

使用宏 NS_SWIFT_NAME 为 Swift 重命名 Objective-C API。这样在混编时在 Swift 中调用到 Objective-C API 的地方就会使用指定的名称，而不是原来的名称，而且这不会影响到 Objective-C 代码，在 Objective-C 中还是使用原来的名称。

该宏在混编时非常有用，因为 Objective-C 的 API 和 Swift 的风格相差比较大，Swift 中的函数命名比较简约，而 OC 通过将方法作用表达得更清晰明了导致方法名较长。这时候就可以使用 NS_SWIFT_NAME 宏来为 Swift 重命名 Objective-C API，优化 Swift 调用体验。

NS_SWIFT_NAME 可用于：

* 类、协议：宏用作前缀
* 枚举、属性、方法或函数、类型别名等其它所有类型：宏用作后缀

### Example

使用 NS_SWIFT_NAME 重命名 Objective-C 类和属性：

```objectivec
NS_SWIFT_NAME(Sandwich.Preferences) // 宏用作前缀
@interface SandwichPreferences : NSObject
  
@property BOOL includesCrust NS_SWIFT_NAME(isCrusty); // 宏用作后缀

@end

@interface Sandwich : NSObject
@end
```

在 Swift 中使用：

```objectivec
// 如果没有重命名 API 的话，是这样调用的
var preferences = SandwichPreferences()
preferences.includesCrust = true
// 重命名了 API 后，调用更优雅
var preferences = Sandwich.Preferences()
preferences.isCrusty = true
```

使用 NS_SWIFT_NAME 重命名 Objective-C 枚举：

```objectivec
typedef NS_ENUM(NSInteger, SandwichBreadType) {
    brioche, pumpernickel, pretzel, focaccia
} NS_SWIFT_NAME(SandwichPreferences.BreadType);
```

### 参考

* [Apple｜Renaming Objective-C APIs for Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/renaming_objective-c_apis_for_swift)