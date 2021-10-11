## 混编｜为 Swift 重命名 Objective-C API（NS_SWIFT_NAME）

使用宏 NS_SWIFT_NAME 为 Swift 重命名 Objective-C API。这样在混编时在 Swift 中调用到 Objective-C API 的地方就会使用指定的名称，而不是原来的名称，而且这不会影响到 Objective-C 代码，在 Objective-C 中还是使用原来的名称，这样就支持一个 API 在两种语言下都有适当的名称。

该宏在混编时非常有用，因为 Objective-C 的 API 和 Swift 的风格相差比较大，Swift 中的函数命名比较简约，而 OC 通过将方法作用表达得更清晰明了导致方法名较长。这时候就可以使用 NS_SWIFT_NAME 宏来为 Swift 重命名 Objective-C API，优化 Swift 调用体验。

NS_SWIFT_NAME 可用于：

* 类、协议（宏用作前缀）
* 枚举、属性、方法或函数、类型别名等其它所有类型（宏用作后缀）

### Example（Apple - Renaming Objective-C APIs for Swift）

#### 重命名 Objective-C 类和属性

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

// Use in Swift
var preferences = SandwichPreferences()
preferences.includesCrust = true
```

使用 NS_SWIFT_NAME 重命名 Objective-C 类和属性：

* 将 SandwichPreferences 变成 Sandwich 的内部类 Preferences
* 给 includesCrust 取个简短的名字

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

// Use in Swift
var preferences = Sandwich.Preferences()
preferences.isCrusty = true
```

#### 重命名 Objective-C 枚举

还可以使用 NS_SWIFT_NAME 重命名 Objective-C 枚举。

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

// Use in Swift
let type = SandwichBreadType.focaccia
```

将 SandwichBreadType 变成 Sandwich.Preferences 的内部枚举。因为前面已经通过使用 NS_SWIFT_NAME 将 SandwichPreferences 变成 Sandwich 的内部类 Preferences，所以这里直接写 `NS_SWIFT_NAME(SandwichPreferences.BreadType)` 即可。

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

// Use in Swift
let type = Sandwich.Preferences.BreadType.focaccia
```

### Example（WWDC20 - 10680 - Refine Objective-C frameworks for Swift）

#### 重命名 Objective-C 方法

为了解决 API 风格上的问题，Swift 会根据一些规则重命名 Objective-C API，例如：

```objectivec
// Declare in Objective-C
- (NSSet<NSString *> *)previousMissionsFlownByAstronaut:(SKAstronaut *)astronaut;

// Generated Swift Interface
open func previousMissionsFlown(by astronaut: SKAstronaut) -> Set<String>
```

在这个例子中，by 变成了参数标签，astronaut 变成了参数名称，方法名缩短为 previousMissionsFlown。

通常这个结果还不错，但这毕竟是计算机的审美结果，很难满足开发者的诉求。

在这个例子中，该方法获取的是某个宇航员以前所执行的任务列表，flown 应该放在参数标签里组成 flownBy 更合适。为了解决这个问题，我们使用 NS_SWIFT_NAME 重命名这个方法。

```objectivec
// Declare in Objective-C
- (NSSet<NSString *> *)previousMissionsFlownByAstronaut:(SKAstronaut *)astronaut NS_SWIFT_NAME(previousMissions(flownBy:));

// Generated Swift Interface
open func previousMissions(flownBy astronaut: SKAstronaut) -> Set<String>
```

#### 重命名 Objective-C 枚举

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

优化点：

1. 针对目前框架，我们有一个 SKFule 类，此时我们可以让这个枚举和 SKFule 联合使⽤，所以我们将其改为 SKFuel.Kind
2. 重命名 SKFuelKindToString 全局函数， 去掉额外的信息，添加⼀个参数标签

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

NS_SWIFT_NAME 处理全局函数的能力还不止这么点。⾸先你可以将 global function 转换成 static method，做法是在 NS_SWIFT_NAME 里指明 Objective—C 的类型并在类型后面使用点语法声明⽅法名。

```objectivec
// Declare in Objective-C
typedef NS_ENUM(NSInteger, SKFuelKind) {
    SKFuelKindH2 = 0,
    SKFuelKindCH4 = 1,
    SKFuelKindC12H26 = 2
} NS_SWIFT_NAME(SKFuel.Kind);

NSString *SKFuelKindToString(SKFuelKind) NS_SWIFT_NAME(SKFuelKind.string(from:));

// Generated Swift Interface
extension SKFuel {
    public enum Kind : Int {
        case H2 = 0
        case CH4 = 1
        case C12H26 = 2
    }
}

extension SKFuel.Kind {
    public static func string(from _: SKFuel.Kind) -> String
}
```

然后，你还可以将其变为实例⽅法！

```objectivec
// Declare in Objective-C
typedef NS_ENUM(NSInteger, SKFuelKind) {
    SKFuelKindH2 = 0,
    SKFuelKindCH4 = 1,
    SKFuelKindC12H26 = 2
} NS_SWIFT_NAME(SKFuel.Kind);

NSString *SKFuelKindToString(SKFuelKind) NS_SWIFT_NAME(SKFuelKind.string(self:));

// Generated Swift Interface
extension SKFuel {
    public enum Kind : Int {      
        case H2 = 0
        case CH4 = 1
        case C12H26 = 2
    }
}

extension SKFuel.Kind {
    public func string() -> String
}
```

最后，你也可以将某个⽅法变为⼀个属性，只需要在前⾯增加⼀个 getter，同理 setter。

```objectivec
// Declare in Objective-C
typedef NS_ENUM(NSInteger, SKFuelKind) {
    SKFuelKindH2 = 0,
    SKFuelKindCH4 = 1,
    SKFuelKindC12H26 = 2
} NS_SWIFT_NAME(SKFuel.Kind);

NSString *SKFuelKindToString(SKFuelKind) NS_SWIFT_NAME(getter:SKFuelKind.description(self:));

// Generated Swift Interface
extension SKFuel {
    public enum Kind : Int {      
        case H2 = 0
        case CH4 = 1
        case C12H26 = 2
    }
}

extension SKFuel.Kind {
    public var description: String { get }
}
```

将这些技术应⽤在充满 C 函数的框架⾥，可以很好的重塑 API，如果你用过 Core Graphics 的话，你会深有体会！

下⾯要说说 `NS_SWIFT_NAME` 的能⼒边界，例如刚才的那个 getter 例子，即使你将⽅法名改为 description，你也⽆法让这个类型遵守 string convertible protocol。

但是我们可以通过在 Swift 文件里添加扩展来使其满足 protocol conformance！

如下图所示，给 SKFuel.Kind 写了⼀个扩展，并使其符合⾃定义字符串转换协议，由于 Objective-C 头文件已经⽤ `NS_SWIFT_NAME` 提供了相应的属性，所以写成这样，我们的 SKFuel.Kind 已经遵守了相应的协议并满足其使用要求。

![](https://images.xiaozhuanlan.com/photo/2020/5f7e4e328f96a4713c2862454e588e88.png)



### Extension 使用 @objc 为 Objective-C 重命名 Swift API





### 参考

* [Apple｜Renaming Objective-C APIs for Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/renaming_objective-c_apis_for_swift)
* [WWDC 内参｜WWDC20 10680 - 让 Objective-C 框架与 Swift 友好共存的秘籍](https://xiaozhuanlan.com/topic/1980624753#sectionobjectivec)

