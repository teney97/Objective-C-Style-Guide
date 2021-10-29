## 混编｜为 Objective-C 添加枚举宏，改善混编体验

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1634001510785-b344fbf4-f5b2-4e6e-a8e1-ffe270fa7ffc.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

关键词：NS_ENUM、NS_OPTIONS、NS_CLOSED_ENUM、NS_TYPED_ENUM、NS_TYPED_EXTENSIBLE_ENUM、NS_STRING_ENUM、NS_EXTENSIBLE_STRING_ENUM、@unknown default

### 前言

使用 Objective-C 的你，是否对 `NS_CLOSED_ENUM`、`NS_STRING_ENUM/NS_EXTENSIBLE_STRING_ENUM`、`NS_TYPED_ENUM/NS_TYPED_EXTENSIBLE_ENUM`  这几个枚举宏感到陌生呢？笔者对修饰 NSNotificationName 的 `NS_EXTENSIBLE_STRING_ENUM` 宏比较好奇，便展开了探索，于是就有了本文。

```objectivec
typedef NSString *NSNotificationName NS_EXTENSIBLE_STRING_ENUM;
UIKIT_EXTERN NSNotificationName const UIApplicationDidEnterBackgroundNotification;
UIKIT_EXTERN NSNotificationName const UIApplicationWillEnterForegroundNotification;
UIKIT_EXTERN NSNotificationName const UIApplicationDidFinishLaunchingNotification;
```

> 在 Xcode 13 中，Apple 已经将其改为使用 `NS_TYPED_EXTENSIBLE_ENUM` 修饰。

### 优雅地声明类型常量枚举

在 Objective-C 中，我们经常会使用 NSString 类型常量来当作 NSDictionary 的 key、NSUserDefaults 的 key、notificationName 等等。例如：

```objectivec
// Dicitonary keys
FOUNDATION_EXTERN NSString * const DCDictionaryKeyTitle;
FOUNDATION_EXTERN NSString * const DCDictionaryKeySubtitle;
FOUNDATION_EXTERN NSString * const DCDictionaryKeyCount;

// 使用
NSDictionary<NSString *, id> *dict = @{......};

NSString *title    = dict[DCDictionaryKeyTitle]; 
NSString *subtitle = dict[DCDictionaryKeySubtitle]; 
NSInteger count    = [dict[DCDictionaryKeyCount] integerValue];
```

这在混编时，在 Swift 中的使用方式为：

```swift
// Objective-C 的常数被自动转换成 Swift 常量
public let DCDictionaryKeyTitle    : String
public let DCDictionaryKeySubtitle : String
public let DCDictionaryKeyCount    : String

// 使用
let dict:[String : Any] = [DCDictionaryKeyTitle    : "a title",
                           DCDictionaryKeySubtitle : "a subTitle",
                           DCDictionaryKeyCount    : 66]

let title    = dict[DCDictionaryKeyTitle]    as! String 
let subtitle = dict[DCDictionaryKeySubtitle] as! String 
let count    = dict[DCDictionaryKeyCount]    as! Int
```

> 你可以查看编译器为 Objective-C 接口生成的 Swift 接口，参考：[Tip：如何查看编译器为 Objective-C 接口生成的 Swift 接口？](https://github.com/teney97/Objective-C-Style-Guide/blob/main/内容/混编｜Tip：如何查看编译器为%20Objective-C%20API%20生成的%20Swift%20API.md)

这样的写法虽然是没有错的，但却存在着问题：

1. dict 的 key 的类型是 String，所以我们其实可以使用任意的字符串当作索引。一般情况下，开发者使用这个 dict 时会去查文件看看有哪些 key 可以使用。但不可避免的是，开发者也能会直接使用字符串如 dict["title"] 来取值，如果不小心拼错的话编译器也不会给警告的，这样就增加了不可预期的错误的风险。
2. 一个小问题，就是代码看起来比较冗长，不符合 Swift 的使用习惯。在 Swift 中我们通常会把这种常量枚举用一个具有字符串原始值的 Enum 或者 Struct 定义，这样我们就能直接使用 `.title` 而不是 `DCDictionaryKeyTitle`，以彰显 Swift 的简洁。

Apple 也发现了这个问题。在 Xcode 8 中，Apple 为 Objective-C 提供了全新的宏 `NS_STRING_ENUM` 和 `NS_EXTENSIBLE_STRING_ENUM`，让字符串类型常量在 Swift 中使用起来更优雅简洁更符合 Swift 的使用习惯。

首先，使用 typedef 对类型常量进行分组，并指定一个类型（如 DCDictionaryKey），涉及到使用该类型常量的地方都改为使用 DCDictionaryKey，而不是 String。然后，在后面添加上宏 `NS_STRING_ENUM`。

```objectivec
typedef NSString *DCDictionaryKey NS_STRING_ENUM;

FOUNDATION_EXTERN DCDictionaryKey const DCDictionaryKeyTitle;
FOUNDATION_EXTERN DCDictionaryKey const DCDictionaryKeySubtitle;
FOUNDATION_EXTERN DCDictionaryKey const DCDictionaryKeyCount;

// 使用
NSDictionary<DCDictionaryKey, id> *dict = @{......};

NSString *title    = dict[DCDictionaryKeyTitle]; 
NSString *subtitle = dict[DCDictionaryKeySubtitle]; 
NSInteger count    = [dict[DCDictionaryKeyCount] integerValue];
```

在 OC 中使用起来没多大变化，但在 Swift 中可就不一样了，真够 Swift！

```swift
// Objective-C 的常数被自动转换成 Swift Struct
public struct DCDictionaryKey : Hashable, Equatable, RawRepresentable {
    public init(rawValue: String)
}
extension DCDictionaryKey {
    public static let title    : DCDictionaryKey
    public static let subtitle : DCDictionaryKey
    public static let count    : DCDictionaryKey
}

// 使用
let dict:[DCDictionaryKey : Any] = [.title    : "a title",
                                    .subtitle : "a subTitle",
                                    .count    : 66]

let title    = dict[.title]    as! String
let subtitle = dict[.subtitle] as! String
let count    = dict[.count]    as! Int

// 这时候如果我们之间使用字符串 "title" 当作 key 的话，编译器会报错
let title    = dict["title"]   as! String // Error: Cannot convert value of type 'String' to expected argument type 'DCDictionaryKey'. Replace '"title"' with 'DCDictionaryKey(rawValue: "title") ?? <#default value#>
```

Foundation 库的 NSNotificationName、NSRunLoopMode 等，或者 SDWebImage 的 SDWebImageContextOption 就是这样处理的。

```objectivec
typedef NSString *NSNotificationName NS_EXTENSIBLE_STRING_ENUM;
UIKIT_EXTERN NSNotificationName const UIApplicationDidEnterBackgroundNotification;
UIKIT_EXTERN NSNotificationName const UIApplicationWillEnterForegroundNotification;
UIKIT_EXTERN NSNotificationName const UIApplicationDidFinishLaunchingNotification;
```

### Objective-C 枚举宏

在 [Apple｜Grouping Related Objective-C Constants](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/grouping_related_objective-c_constants) 中，Apple 详细列举了 `NS_ENUM`、`NS_CLOSED_ENUM`、`NS_OPTIONS`、`NS_TYPED_ENUM`、`NS_TYPED_EXTENSIBLE_ENUM` 等宏的使用场景，用好它们以改善在混编时在 Swift 中的编程体验。另外，Apple 建议弃用 `NS_STRING_ENUM/NS_EXTENSIBLE_STRING_ENUM` 而改用 `NS_TYPED_ENUM/NS_TYPED_EXTENSIBLE_ENUM`。

* NS_ENUM：用于简单的枚举
* NS_CLOSED_ENUM：用于不会变更枚举成员的简单的枚举（简称 “冻结枚举” ）
* NS_OPTIONS：用于可选类型枚举
* NS_TYPED_ENUM：用于类型常量枚举
* NS_TYPED_EXTENSIBLE_ENUM：用于可扩展的类型常量枚举

#### NS_ENUM

用于声明简单的枚举，这个大家都很熟悉了。

```objectivec
// Declare in Objective-C
typedef NS_ENUM(NSInteger, UITableViewCellStyle) {
    UITableViewCellStyleDefault,
    UITableViewCellStyleValue1,
    UITableViewCellStyleValue2,
    UITableViewCellStyleSubtitle
};

// In Swift, the UITableViewCellStyle enumeration is imported like this:
public enum UITableViewCellStyle : Int {
    case `default` = 0
    case value1 = 1
    case value2 = 2
    case subtitle = 3
}

// Use in Swift
let style = UITableViewCellStyle.default
```

这个知识点看似没用，实则大大有用。在 Objective-C 中，除了使用 NS_ENUM 宏，还可以像如下等方式定义枚举。它或许是你或同事的编码习惯，又或许是历史遗留的代码。虽然这样的写法并没有错，但 Generated Swift Interface 却不尽如人意，导致在 Swift 中使用时只能使用原始的完整的枚举名称。

```objectivec
// Declare in Objective-C
typedef enum: NSUInteger {
    UITableViewCellStyleDefault,
    UITableViewCellStyleValue1,
    UITableViewCellStyleValue2,
    UITableViewCellStyleSubtitle
} UITableViewCellStyle;

// Generated Swift Interface
public struct UITableViewCellStyle : Equatable, RawRepresentable {
    public init(_ rawValue: UInt)
    public init(rawValue: UInt)
    public var rawValue: UInt
}

// Use in Swift
let style = UITableViewCellStyleDefault
```

在 [《Effective Objective-C 2.0》5. 用枚举表示状态、选项、状态码](https://juejin.cn/post/6904440708006936590#heading-6) 中也提到了使用 NS_ENUM 和 NS_OPTIONS 来声明枚举类型的优点。如果你的工程处于混编阶段，不妨将 Objective-C 中的枚举类型改为 NS_ENUM 和 NS_OPTIONS 声明，以优化 Swift 编程体验吧。

#### NS_CLOSED_ENUM

用于声明不会变更枚举成员的简单的枚举（简称 “冻结枚举” ），对应 Swift 中的 `@frozen` 关键字。冻结枚举对于希望在 switch 语句中匹配有限状态集的时候非常有用，这个有限状态集是一个完整的集合，覆盖了所有情况，将来不会再有其他新的情况。

例如，NSComparisonResult 枚举用于指定如何排序，在两个数比大小时，无非就 <、=、> 三种情况，所以非常适合使用冻结枚举。

```objectivec
// Declare in Objective-C
typedef NS_CLOSED_ENUM(NSInteger, NSComparisonResult) {
    NSOrderedAscending = -1L,
    NSOrderedSame,
    NSOrderedDescending
};

// In Swift, the NSComparisonResult enumeration is imported like this:
@frozen public enum NSComparisonResult : Int {
    case orderedAscending = -1
    case orderedSame = 0
    case orderedDescending = 1
}
```

> 使用 NS_ENUM 和 NS_CLOSED_ENUM 枚举宏在导入到 Swift 时生成的是实际 Enum 类型，而其它枚举宏都是生成 Struct 类型。

相比较于非冻结枚举，冻结枚举降低了灵活性，但提升了性能。一旦枚举被标记为冻结枚举，那么在未来版本的库中就不能通过添加、删除或重新排序枚举的 case，否则会破坏 ABI 兼容性。

**Swift 中的 default 与 @unknown default**

* 对于非冻结枚举，你需要使用 `default` 或者 `@unknown default` 来处理未知的 case（未来可能新增枚举类型），否则会得到编译器警告 `Switch covers known cases, but 'enumType' may have additional unknown values`，但 Xcode 的 fix 方案是使用 `@unknown default` 。
* 而对于冻结枚举，使用 `@unknown default` 无论如何都会得到编译器警告。
  * 如果你穷举了所有 case，将得到警告 `Case is already handled by previous patterns; consider removing it`，因为冻结枚举已经约定好将来不会添加新的枚举成员，所以 `@unknown default` case 永远不会执行。虽然这里使用 `default` 不会得到警告，但也是不会执行的。
  * 如果你没有穷举所有 case，将得到警告 `Switch must be exhaustive`，使用  `@unknown default` 必须穷举所有 case。

简单来说 `default` 和 `@unknown default` 都可以用来处理已知以及未知的情况。区别在于，使用 `@unknown default`，如果你没有穷举所有枚举类型，或者未来有新增枚举类型，那么编译器会给出警告提示。关于选择应该是，对于非冻结枚举，如果你想穷举所有 case，并希望未来有新增枚举类型时得到编译器警告，那么就使用 `@unknown default`。也就是说，`@unknown default` 应该只匹配未来加入的枚举 `case`。

**关于冻结枚举与非冻结枚举，可参阅：**

* [Apple｜SE-0192 Handling Future Enum Cases](https://github.com/apple/swift-evolution/blob/master/proposals/0192-non-exhaustive-enums.md)
* [SwiftGG｜对未来枚举的 case 进行 switch](https://swiftgg.gitbook.io/swift/yu-yan-can-kao/05_statements#branch-statements)
* [SwiftGG｜frozen](https://swiftgg.gitbook.io/swift/yu-yan-can-kao/07_attributes#frozen)
* [Swift 5 Frozen enums](https://useyourloaf.com/blog/swift-5-frozen-enums/)
* [Swiftjectivec｜NS_CLOSED_ENUM](https://www.swiftjectivec.com/ns_closed_enum/)

#### NS_OPTIONS

用于声明可选类型枚举，这个大家也都很熟悉了。需要注意的地方在上文 **NS_ENUM** 中已经提到了，尽量使用 NS_OPTIONS 来定义选项枚举。

```objectivec
// Declare in Objective-C
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};

// In Swift, the UIViewAutoresizing type is imported like this:
public struct UIViewAutoresizing: OptionSet {
    public init(rawValue: UInt)
    
    public static var flexibleLeftMargin: UIViewAutoresizing { get }
    public static var flexibleWidth: UIViewAutoresizing { get }
    public static var flexibleRightMargin: UIViewAutoresizing { get }
    public static var flexibleTopMargin: UIViewAutoresizing { get }
    public static var flexibleHeight: UIViewAutoresizing { get }
    public static var flexibleBottomMargin: UIViewAutoresizing { get }
}

// Use in Swift
let style = UIViewAutoresizing([.flexibleWidth, .flexibleHeight])
```

#### NS_TYPED_ENUM

用于声明类型常量枚举，不局限于字符串类型常量，NS_STRING_ENUM 可以用它替代。

可以使用 typedef 对类型常量进行分组，并指定一个类型（如下 TrafficLightColor），然后在后面添加上宏 NS_TYPED_ENUM。

>使用 NS_STRING_ENUM 宏，在逻辑上你不能在 Swift 中使用 extension 扩展新的常量集，虽然这是允许的。如果你需要做此支持，请使用 NS_TYPED_EXTENSIBLE_ENUM。

```swift
// Store the three traffic light color options as 0, 1, and 2.
typedef long TrafficLightColor NS_TYPED_ENUM;
 
FOUNDATION_EXTERN TrafficLightColor const TrafficLightColorRed;
FOUNDATION_EXTERN TrafficLightColor const TrafficLightColorYellow;
FOUNDATION_EXTERN TrafficLightColor const TrafficLightColorGreen;

// In Swift, the TrafficLightColor type is imported like this:
public struct TrafficLightColor : Hashable, Equatable, RawRepresentable {
    public init(rawValue: Int)
}
extension TrafficLightColor {
    public static let red: TrafficLightColor
    public static let yellow: TrafficLightColor
    public static let green: TrafficLightColor
}

// Use in Swift
let color = TrafficLightColor.red
```

#### NS_TYPED_EXTENSIBLE_ENUM

用于声明可扩展的类型常量枚举。在导入到 Swift 中时，它生成的 Struct 与 NS_TYPED_ENUM 生成的 Struct 的区别在于多了一个初始化器。

```swift
// declared
typedef long FavoriteColor NS_TYPED_EXTENSIBLE_ENUM;
FOUNDATION_EXTERN FavoriteColor const FavoriteColorBlue;

// imported
public struct FavoriteColor : Hashable, Equatable, RawRepresentable {
    public init(_ rawValue: Int)
    public init(rawValue: Int)
}
extension FavoriteColor {
    public static let blue: FavoriteColor
}

// extended
extension FavoriteColor {
    static var green: FavoriteColor {
        return FavoriteColor(1) // blue is 0, green is 1, and new favorite colors could follow
    }
}
```

最后，让我们看一下 `NS_STRING_ENUM/NS_EXTENSIBLE_STRING_ENUM`、`NS_TYPED_ENUM/NS_TYPED_EXTENSIBLE_ENUM` 的宏定义，它们的替换宏都为 `_NS_TYPED_ENUM/_NS_TYPED_EXTENSIBLE_ENUM`。我们优先使用 `NS_TYPED_ENUM/NS_TYPED_EXTENSIBLE_ENUM` 以保持代码统一性。

```objectivec
#define _NS_TYPED_ENUM _CF_TYPED_ENUM
#define _NS_TYPED_EXTENSIBLE_ENUM _CF_TYPED_EXTENSIBLE_ENUM

// Note: NS_TYPED_ENUM is preferred to NS_STRING_ENUM
#define NS_STRING_ENUM _NS_TYPED_ENUM
// Note: NS_TYPED_EXTENSIBLE_ENUM is preferred to NS_EXTENSIBLE_STRING_ENUM
#define NS_EXTENSIBLE_STRING_ENUM _NS_TYPED_EXTENSIBLE_ENUM

#define NS_TYPED_ENUM _NS_TYPED_ENUM
#define NS_TYPED_EXTENSIBLE_ENUM _NS_TYPED_EXTENSIBLE_ENUM
```

### 小结

通过阅读本文，你是否对 Objective-C 的枚举宏有了进一步的了解呢？用好它们以改善在混编时在 Swift 中的编程体验。使用 `NS_ENUM` 和 `NS_OPTIONS` 来声明简单枚举和选项枚举，以优化 Swift 编程体验。`NS_CLOSED_ENUM` 用于声明不会变更枚举成员的冻结枚举，对应 Swift 中的 `@frozen` 关键字，以降低灵活性的代价，换取了性能上的提升。`NS_STRING_ENUM/NS_EXTENSIBLE_STRING_ENUM`、`NS_TYPED_ENUM/NS_TYPED_EXTENSIBLE_ENUM` 用于声明字符串常量/类型常量枚举，这在混编时在 Swift 中使用起来更简洁优雅更符合 Swift 的使用习惯。 

### 参考

* [Apple｜Grouping Related Objective-C Constants](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/grouping_related_objective-c_constants)
* [Apple｜macOS Mojave 10.14 Release Notes - Foundation Release Notes](https://developer.apple.com/documentation/macos-release-notes/foundation-release-notes#3035774)
* [Apple｜SE-0192 Handling Future Enum Cases](https://github.com/apple/swift-evolution/blob/master/proposals/0192-non-exhaustive-enums.md)
* [Apple｜WWDC20 - Refine Objective-C frameworks for Swift](https://developer.apple.com/videos/play/wwdc2020/10680/)
* [SwiftGG｜对未来枚举的 case 进行 switch](https://swiftgg.gitbook.io/swift/yu-yan-can-kao/05_statements#branch-statements)
* [SwiftGG｜frozen](https://swiftgg.gitbook.io/swift/yu-yan-can-kao/07_attributes#frozen)
* [Swift 5 Frozen enums](https://useyourloaf.com/blog/swift-5-frozen-enums/)
* [Swiftjectivec｜NS_CLOSED_ENUM](https://www.swiftjectivec.com/ns_closed_enum/)
* [Dcard Lab｜初探 NS_STRING_ENUM](https://medium.com/dcardlab/%E5%88%9D%E6%8E%A2-ns-string-enum-7fc408596802)

