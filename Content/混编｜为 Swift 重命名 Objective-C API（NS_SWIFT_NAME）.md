## 混编｜为 Swift 重命名 Objective-C API（NS_SWIFT_NAME）

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1634022274579-75656438-4c8b-431a-b1d5-d4593c347aef.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

关键词：NS_SWIFT_NAME、@objc

### 前言

Swift 和 Objective-C 的 API 命名规范有些不同，例如：

1. Swift 的方法由基名和参数标签组成，而 Objective-C 则只有参数标签，没有单独的基名，所以基名的信息会包含在第一个参数标签里；
2. Swift 的参数标签有时会省略从类型上即可看出的明显信息，而 Objective-C 一般不遵循此规则。

因此，Swift 的方法命名比较简约。而 Objective-C 为了将方法作用表达得清晰明了，所以方法名显得略长一些，可以看看[《Effective Objective-C 2.0》19. 使用清晰而协调的命名方式](https://juejin.cn/post/6904440708006936590#heading-22)。这样就导致在混编时，一个 API 没办法保证在两种语言中都有合适的名称。

为了解决这一问题，Swift 会根据一些[规则](https://github.com/apple/swift/blob/main/docs/CToSwiftNameTranslation.md)重命名导入的 Objective-C API，例如把对应类型名称的前缀和后缀去掉，并根据英文语法和词汇表来将 Objective-C 的 SEL 的第一部分分割为基名和参数标签；反之，Objective-C 也会根据一些规则重命名导入的 Swift API。通常这个结果还不错，但这毕竟是计算机的审美结果，有时会不尽如人意。

为此，Apple 于 [Xcode 7](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW326) 中引入了宏 `NS_SWIFT_NAME`。我们可以使用该宏为 Swift 重命名 Objective-C API，来支持在 Swift 中以自定义名称使用 Objective-C API，而且在 Objective-C 中保留了原始名称。除此之外，我们还可以通过 `@objc` 为  Objective-C 重命名 Swift API。这样就支持一个 API 在两种语言中都有合适的名称。在本篇文章中，我们就通过 Apple 举的一些例子来看看 `NS_SWIFT_NAME` 如何使用。

### 使用宏 NS_SWIFT_NAME 为 Swift 重命名 Objective-C API

`NS_SWIFT_NAME` 可用于：

* 类、协议（宏用作前缀）
* 枚举、属性、方法或函数、类型别名等其它所有类型（宏用作后缀）

#### Example（Apple - Xcode Release Notes - Xcode 7）

##### 重命名枚举的 case

正常来说，以下 Objective-C 枚举导入到 Swift 中时，会去掉完整前缀 `DisplayMode`。但 DisplayMode256Color 去掉该前缀后就是以数字开头了，这是不允许的。因此编译器会为所有 case 保留枚举类型名称的最后一个单词，改正 DisplayMode256Color 在 Swift 中的名称的同时，保持所有 case 命名的一致性，这里就是保留 `mode`。

```objectivec
// Objective-C Interface
typedef NS_ENUM(NSInteger, DisplayMode) {
    DisplayMode256Color,
    DisplayModeThousandsOfColors,
    DisplayModeMillionsOfColors
};

// Generated Swift Interface
enum DisplayMode : Int {
    case mode256Colors = 0
    case modeThousandsOfColors = 1
    case modeMillionsOfColors = 2
}
```

我们可以针对 DisplayMode256Color case 做下优化，使用 `NS_SWIFT_NAME` 宏为其在 Swift 中重命名，不以数字开头。这样所有 case 在导入到 Swift 中时，都会去掉完整的前缀 `DisplayMode` 了。

```objectivec
// Objective-C Interface
typedef NS_ENUM(NSInteger, DisplayMode) {
    DisplayMode256Colors NS_SWIFT_NAME(with256Colors),
    DisplayModeThousandsOfColors,
    DisplayModeMillionsOfColors,
};

// Generated Swift Interface
enum DisplayMode : Int {
    case with256Colors = 0
    case thousandsOfColors = 1
    case millionsOfColors = 2
}
```

##### 将 Objective-C 工厂方法作为构造器导入到 Swift

以下工厂方法的命名不满足自动转换为构造器导入到 Swift 的约定，需要以 `controller` 开头比如命名为 `controllerForURLKind:` 才行。不过我们可以通过 `NS_SWIFT_NAME` 将其作为构造器导入到 Swift。

```objectivec
// Objective-C Interface
@interface MyController : UIViewController
+ (instancetype)standardControllerForURLKind:(URLKind)kind NS_SWIFT_NAME(init(URLKind:));
@end
  
// Generated Swift Interface
class MyController: UIViewController {
    convenience init(URLKind kind: URLKind)
}
```

#### Example（Apple - Renaming Objective-C APIs for Swift）

##### 重命名 Objective-C 类、属性

我们先来看一下优化前的 Objective-C 接口：

```objectivec
// Objective-C Interface
@interface SandwichPreferences : NSObject
@property BOOL includesCrust;
@end

@interface Sandwich : NSObject
@end
  
// Generated Swift Interface
open class SandwichPreferences : NSObject {
    open var includesCrust: Bool
}

open class Sandwich : NSObject {
}

// Use it in Swift
var preferences = SandwichPreferences()
preferences.includesCrust = true
```

使用 `NS_SWIFT_NAME` 重命名 Objective-C 类和属性：

* 将 SandwichPreferences 变成 Sandwich 的内部类 Preferences，使其附属于 Sandwich
* 给 includesCrust 换个名字 isCrusty，更简短了

```objectivec
// Objective-C Interface
NS_SWIFT_NAME(Sandwich.Preferences) // 宏用作前缀
@interface SandwichPreferences : NSObject
@property BOOL includesCrust NS_SWIFT_NAME(isCrusty); // 宏用作后缀
@end

@interface Sandwich : NSObject
@end
 
// Generated Swift Interface
extension Sandwich {
    open class Preferences : NSObject {
        open var isCrusty: Bool
    }
}

open class Sandwich : NSObject {
}

// Use it in Swift
var preferences = Sandwich.Preferences()
preferences.isCrusty = true
```

##### 重命名 Objective-C 枚举

优化前：

```objectivec
// Declare in Objective-C
typedef NS_ENUM(NSInteger, SandwichBreadType) {
    brioche, pumpernickel, pretzel, focaccia
};

// Generated Swift Interface
public enum SandwichBreadType : Int {
    case brioche = 0
    case pumpernickel = 1
    case pretzel = 2
    case focaccia = 3
}

// Use it in Swift
let type = SandwichBreadType.focaccia
```

将 SandwichBreadType 变成 Sandwich.Preferences 的内部枚举 BreadType，注意这里需要写 Objective-C 中的名称 SandwichPreferences。

优化后：

```objectivec
// Declare in Objective-C
typedef NS_ENUM(NSInteger, SandwichBreadType) {
    brioche, pumpernickel, pretzel, focaccia
} NS_SWIFT_NAME(SandwichPreferences.BreadType);

// Generated Swift Interface
extension Sandwich.Preferences {
    public enum BreadType : Int { 
        case brioche = 0
        case pumpernickel = 1
        case pretzel = 2
        case focaccia = 3
    }
}

// Use it in Swift
let type = Sandwich.Preferences.BreadType.focaccia
```

#### Example（WWDC20 - 10680 - Refine Objective-C frameworks for Swift）

##### 重命名 Objective-C 方法

```objectivec
// Declare in Objective-C
- (NSSet<NSString *> *)previousMissionsFlownByAstronaut:(SKAstronaut *)astronaut;

// Generated Swift Interface
open func previousMissionsFlown(by astronaut: SKAstronaut) -> Set<String>
```

在这个例子中，Objective-C API 在导入到 Swift 中时就根据规则重命名了，by 变成了参数标签，astronaut 变成了参数名称，基名缩短为 previousMissionsFlown。

这个结果还不错，但是一些开发人员会觉得该 Objective-C 的 SEL 没有得到正确的分割，由于该方法获取的是某个宇航员以前所执行的任务列表，所以 flown 应该放在参数标签里组成 flownBy 更合适。

为了解决这个问题，我们可以使用 `NS_SWIFT_NAME` 重命名这个方法，进行微调。

```objectivec
// Declare in Objective-C
- (NSSet<NSString *> *)previousMissionsFlownByAstronaut:(SKAstronaut *)astronaut NS_SWIFT_NAME(previousMissions(flownBy:));

// Generated Swift Interface
open func previousMissions(flownBy astronaut: SKAstronaut) -> Set<String>
```

##### 重命名 Objective-C 枚举、全局函数

```objectivec
// Declare in Objective-C
typedef NS_ENUM(NSInteger, SKFuelKind) {
    SKFuelKindH2 = 0,
    SKFuelKindCH4 = 1,
    SKFuelKindC12H26 = 2
};

NSString *SKFuelKindToString(SKFuelKind);

// Generated Swift Interface
public enum SKFuelKind : Int {
    case H2 = 0
    case CH4 = 1
    case C12H26 = 2
}

public func SKFuelKindToString(_: SKFuelKind) -> String
```

在这个例子中，Generated Swift Interface 已经挺不错了。例如使用 NS_ENUM 来声明 Objective-C 枚举，这样在导入到 Swift 中时就会变成 enum 类型，可以看看 [iOS 混编｜为 Objective-C 添加枚举宏，改善混编体验 —— NS_ENUM](https://juejin.cn/post/6999460035508043807#heading-3)。

不过，还是存在可以改进的地方：

1. 假如我们有一个 SKFule 类，那么我们就可以让这个枚举附属于 SKFule，将其改为 SKFuel.Kind
2. `NS_SWIFT_NAME` 还可以用于全局函数、常量、变量等。这里我们可以重命名 SKFuelKindToString 全局函数为 string， 去掉基名中多余的信息，并添加⼀个参数标签 from

```objectivec
// Declare in Objective-C
typedef NS_ENUM(NSInteger, SKFuelKind) {
    SKFuelKindH2 = 0,
    SKFuelKindCH4 = 1,
    SKFuelKindC12H26 = 2
} NS_SWIFT_NAME(SKFuel.Kind);

NSString *SKFuelKindToString(SKFuelKind) NS_SWIFT_NAME(string(from:));

// Generated Swift Interface
extension SKFuel {
    public enum Kind : Int {
        case H2 = 0
        case CH4 = 1
        case C12H26 = 2
    }
}

public func string(from _: SKFuel.Kind) -> String
```

有意思的是，`NS_SWIFT_NAME` 在处理全局常量、变量，特别是在处理全局函数时，它的能力更加强大，能够极大程度地改变 API。

⾸先，我们可以将 `全局函数` 转变为 `静态方法`，做法是通过在 `NS_SWIFT_NAME` 里指定类型的 Objective-C 名称，然后加一个点，再加上静态方法的名称。

```objectivec
// Declare in Objective-C
NSString *SKFuelKindToString(SKFuelKind) NS_SWIFT_NAME(SKFuelKind.string(from:));

// Generated Swift Interface
extension SKFuel.Kind {
    public static func string(from _: SKFuel.Kind) -> String
}
```

接着，我们还可以将其转变为 `实例⽅法`，做法是把其中实例类型参数的参数标签改为 self，这样一来 Swift 也就知道在原全局函数的哪里传入当前实例。例如这里就是将 from 改为 self：

```objectivec
// Declare in Objective-C
NSString *SKFuelKindToString(SKFuelKind) NS_SWIFT_NAME(SKFuelKind.string(self:));

// Generated Swift Interface
extension SKFuel.Kind {
    public func string() -> String
}
```

> 这里再写个 Demo 进一步解释一下，把其中实例类型参数的参数标签改为 self，这样一来 Swift 也就知道在原全局函数的哪里传入当前实例。
>
> ```objectivec
> // Declare in Objective-C
> NSString *SKFuelKindToString(NSArray *, SKFuelKind, NSString *) NS_SWIFT_NAME(SKFuelKind.string(array:self:string:));
> 
> // Generated Swift Interface
> extension SKFuel.Kind {
>     public func string(array _: [Any], string _: String) -> String
> }
> ```

然后，我们还可以将这个⽅法转变为⼀个 `实例属性`，只需要在前⾯添加 `getter:`，setter 同理。以下 description 是指定的属性名称。

```objectivec
// Declare in Objective-C
NSString *SKFuelKindToString(SKFuelKind) NS_SWIFT_NAME(getter:SKFuelKind.description(self:));

// Generated Swift Interface
extension SKFuel.Kind {
    public var description: String { get }
}
```

`NS_SWIFT_NAME` 还可以用于与 Swift 类型名称完全不一样的库中，例如在 C 库中的小写类型名称。

将以上这些技巧应⽤在充满函数的框架⾥，可以极大程度地重塑 API 风格。如果你在 Objective-C 和 Swift 里都用过 Core Graphics 的话，你会深有体会。Apple 称其把 `NS_SWIFT_NAME` 用在了数百个全局函数上，将它们转换为方法、属性和构造器，以更加方便地在 Swift 中使用。

`NS_SWIFT_NAME` 也有能力不足之处。例如在上个例子中，即使我们将全局函数转变为了实例属性 description，但你无法使用 `NS_SWIFT_NAME` 来让类型遵守 [CustomStringConvertible](https://developer.apple.com/documentation/swift/customstringconvertible) 协议。

但是我们可以在 Swift 中通过一行代码，就是扩展 SKFuel.Kinds 来使其遵守 CustomStringConvertible 协议。

```swift
extension SKFuel.Kinds: CustomStringConvertible {}
```

### NS_SWIFT_NAME 无法使用的场景

除了以上提到的问题：即使我们将全局函数转变为了实例属性 description，但你无法使用 `NS_SWIFT_NAME` 来让类型遵守 CustomStringConvertible 协议。

笔者在实践过程中还遇到了一个 `NS_SWIFT_NAME` 无法使用的场景：在 Objective-C 协议中将工厂方法转变为构造器导入到 Swift 中。在 Objective-C 协议中尝试转变为构造器的工厂方法，在 Generated Swift Interface 中不会生成，而且使用会报错 `Argument passed to call that takes no arguments`，奇怪的地方是调用的时候编译器明明给出了代码补全提示却不让用。

```objectivec
@protocol iAnimationView <NSObject>
+ (instancetype)animationNamed:(NSString *)animationName type:(AnimationType)type NS_SWIFT_NAME(init(named:type:));
@end
  
@interface AnimationView : UIView<iAnimationView>
@end
  
// Use it in Swift
let view = AnimationView(named: name, type: type, bundle: bundle) // Error: Argument passed to call that takes no arguments
```

解决方案就是将尝试转变为构造器的工厂方法在类的声明中再写一遍：

```objectivec
@protocol iAnimationView <NSObject>
+ (instancetype)animationNamed:(NSString *)animationName type:(AnimationType)type;
@end
  
@interface AnimationView : UIView<iAnimationView>
+ (instancetype)animationNamed:(NSString *)animationName type:(AnimationType)type NS_SWIFT_NAME(init(named:type:));
@end
  
// Use it in Swift
let view = AnimationView(named: name, type: type, bundle: bundle) // Okay
```

### Extension：为 Objective-C 重命名 Swift API

我们可以在 projectName-Swift.h 文件中查看编译器为 Swift Interface 生成的 Objective-C Interface，编译器也会根据一些规则为 Objective-C 重命名 Swift API，通常这个结果也还不错，但有时候还是存在优化空间的，这时候我们也可以自定义重命名 Swift API。

我们再拿上文 “获取某个宇航员以前所执行的任务列表的方法” 的例子，假如这个方法定义在 Swift 类中：

```swift
@objc func previousMissions(flownBy astronaut: SKAstronaut) -> Set<String> { ... }
```

生成的 Objective-C API：

```objectivec
// projectName-Swift.h
- (NSSet<NSString *> * _Nonnull)previousMissionsWithFlownBy:(SKAstronaut * _Nonnull)astronaut SWIFT_WARN_UNUSED_RESULT;
```

方法名不够清晰，这不太符合 Objective-C API 命名规范，我们期望的转换结果如下：

```objectivec
- (NSSet<NSString *> * _Nonnull)previousMissionsFlownByAstronaut:(SKAstronaut * _Nonnull)astronaut;
```

这时候我们可以为 Objective-C 重命名该 Swift API：

```swift
@objc(previousMissionsFlownByAstronaut:) func previousMissions(flownBy astronaut: SKAstronaut) -> Set<String> { ... }
```

### 小结

Swift 和 Objective-C 的 API 命名规范有些不同，在混编时，虽然编译器会根据一些规则重命名 Objective-C 与 Swift API 且通常结果还不错，但这毕竟是计算机的审美结果，有时会不尽如人意。本篇文章讲解了如何自定义重命名 Objective-C 与 Swift API，掌握它们就可以人为地优化重命名的 API，提升混编体验。

### 参考

* [Apple｜Xcode Release Notes - Xcode 7](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW326)
* [Apple｜Renaming Objective-C APIs for Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/renaming_objective-c_apis_for_swift)
* [Apple｜CToSwiftNameTranslation](https://github.com/apple/swift/blob/main/docs/CToSwiftNameTranslation.md)
* [Apple｜WWDC20 10680 - Refine Objective-C frameworks for Swift](https://developer.apple.com/videos/play/wwdc2020/10680/)
* [WWDC 内参｜WWDC20 10680 - 让 Objective-C 框架与 Swift 友好共存的秘籍](https://xiaozhuanlan.com/topic/1980624753#sectionobjectivec)

