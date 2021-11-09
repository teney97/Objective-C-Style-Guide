## 混编｜为 Objective-C API 指定可空性

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1636303863964-b3d0e51e-69f7-44f4-aa60-c703bf4699db.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

关键词：Swift 可选类型、Objective-C nullability annotations、nullable、nonnull、null_resettable、null_unspecified、NS_ASSUME_NONNULL_BEGIN/NS_ASSUME_NONNULL_END

字数：4100+

### 前言

使用 nullability annotations 为 Objective-C API 指定可空性，以控制 Objective-C 声明中的指针类型如何导入到 Swift 中，让混编更安全更顺利；而且可以让开发者平滑地从 Objective-C 过渡到 Swift；还让 Objective-C API 更具表现力且更易于正确使用，促使开发者在编写 Objective-C 代码时更加规范，减少同事之间的沟通成本。

### nullability annotations 缘由

在 WWDC2014 上，Apple 发布了 Swift 和 [Xcode 6](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW453)。Swift 完美支持以下，这意味着 Swift 和 Objective-C 可以混编。

- Cocoa and Cocoa touch
- Build with LLVM compiler
- Optimizer and Autovectorizer
- ARC memory management
- Same runtime as Objective-C

为了更好的混编，Apple 于 [Xcode 6.3](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW378) 引入了 nullability annotations 特性，该特性是为了解决什么问题呢？

Swift 增加了可选（Optional）类型（如 `String?`），用于处理值可能缺失的情况。可选类型表示两种情况：1）有值；2）nil，也就是没有值。一个 non-optional String 写作 `String`，而一个 optional String 写作 `String?`。Swift 是类型安全的，如果你的代码需要一个 `String`，类型安全会阻止你意外传入一个 `String?`。

而 Objective-C 中没有可选类型，`NSString *` 既可以表示这个类型是 optional，也可以表示是 non-optional。这样在混编时就产生了一个问题，Swift 编译器并不知道一个 Objective-C 指针类型到底是 optional 还是 non-optional。在这种情况下，编译器会将 Objective-C 的指针类型当成是 `隐式解析可选类型`（例如 `String!`）导入到 Swift 中。

```swift
// Objective-C Interface
@interface MyList : NSObject
- (MyListItem *)itemWithName:(NSString *)name;
- (NSString *)nameForItem:(MyListItem *)item;
@property (copy) NSArray<MyListItem *> *allItems;
@end

// Generated Swift Interface
class MyList : NSObject {
    func item(withName name: String!) -> MyListItem!
    func name(for item: MyListItem!) -> String!
    var allItems: [MyListItem]!
}
```

这从而引发出一些问题，比如：

1. 如果在 Swift 中调用 `list.item(withName: name).listItemMethod()` 而 `itemWithName:` 方法返回了一个 nil 的话，将会因为隐式解析一个为 nil 的可选值而得到运行时错误，导致 Crash，这让混编变得不安全。
2. 除混编之外，由于 Objective-C 不支持可选类型，我们经常需要对对象做非空判断，以避免发生异常，比如调用一个值为 nil 的 block 将导致 Crash。假如你写了一个方法，你希望调用者传递进来的是一个非空的对象，你却只能通过注释告诉他，而不能通过编译器的能力。这可能还不够，你还需要自己做空值处理，以避免调用者意外地传来一个 nil 导致异常的发生。

因此，Apple 为 Objective-C 引入了 nullability annotations 特性，为 Objective-C API 指定可空性，可以控制 Objective-C 声明中的指针类型如何导入到 Swift 中（以 optional 还是 non-optional），让混编更安全；同时，也给 Objective-C 新增了 nullability 类型警告，让 Objective-C 代码更加规范。

### nullability annotations 使用

Apple 于 [Xcode 6.3](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW378) 引入了 nullability annotations 特性 —— 一组可空性限定符，可用于任意指针类型中（包括 Objective-C 指针、C 指针、C++ 成员指针等等）。在混编时，可空性限定符会影响 Objective-C API 导入到 Swift 时的可选性。非空（nonnull）限定类型作为 non-optional（如 String）导入，可空（nullable）限定类型作为 optional（如 String?）导入，而不是被编译器默认作为 `隐式解析可选类型` （如 String!）导入。

```swift
// Objective-C Interface
@interface MyList : NSObject
- (nullable MyListItem *)itemWithName:(nonnull NSString *)name;
- (nullable NSString *)nameForItem:(nonnull MyListItem *)item;
@property (copy, nonnull) NSArray<MyListItem *> *allItems;
@end

// Generated Swift Interface
class MyList: NSObject {
    func item(withName name: String) -> MyListItem?
    func name(for item: MyListItem) -> String?
    var allItems: [MyListItem]
}
```

以上是可空性限定符的不带下划线的版本（如 `nullable`），它还有双下划线版本（如 `__nullable`），用在 C 指针、C++ 成员指针、Block 指针、Block 返回值与参数的指针中。

```swift
// C API
void enumerateStrings(__nonnull CFStringRef (^ __nullable callback)(void));
// Generated Swift Interface
func enumerateStrings(_ callback: (() -> Unmanaged<CFString>)?)
```

> 由于与第三方库的潜在冲突，Apple 在 [Xcode 7](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW326) 中将双下划线版本（`__nullable、__nonnull、__null_unspecified`）更名为单下划线以大写字母开头的版本（`_Nullable、_Nonnull、_Null_unspecified`）。但是，为了与 Xcode 6.3 兼容，编译器预定义了双下划线版本的宏来映射到新名称。加上没有下划线的（`nullable、nonnull、null_unspecified`）总共有 3 种版本。

| 可空性限定符                                             | 导入到 Swift 中               | 意义                                                         |
| -------------------------------------------------------- | ----------------------------- | ------------------------------------------------------------ |
| nullable、_Nullable 、__nullable                         | optional，如 String？         | 该值可以是 nil                                               |
| nonnull、_Nonnull、__nonnull                             | non-optional，如 String       | 该值永远不会为 nil                                           |
| null_unspecified、_Null_unspecified 、__null_unspecified | 隐式解析 optional，如 String! | 未指定值是否可以 nil（非常罕见，除非将其作为过渡工具，否则应避免使用）。使用该限定符的结果和不使用 nullability annotations 特性的结果是一样的，我认为该限定符的作用是在过渡时抵消 audited Regions |
| null_resettable（只用于属性）                            | 隐式解析 optional，如 String! | 1. 属性的 setter 允许设置 nil 以将值重置为某个默认值，但其 getter 永远不会返回 nil（因为提供了一个默认值）； 2. 必须重写 setter 或 getter 做非空处理。否则会报警告 `Synthesized setter 'setName:' for null_resettable property 'name' does not handle nil`，同时存在安全隐患 |

> 注意：nullability annotations 不能用于非指针类型，因为 Objective-C 中 nil 只能用在引用对象的指针上，而对于基础数据类型如 NSInteger 对于没有值的情况我们一般是使用 NSNotFound。在 [混编｜为 Swift 改进 Objective-C API 声明（NS_REFINED_FOR_SWIFT）](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/混编｜为%20Swift%20改进%20Objective-C%20API%20声明（NS_REFINED_FOR_SWIFT）.md) 有讲解如何在 Swift 中将 Objective-C 的 NSNotFound 改进为 nil 的例子，可以看看。

#### 使用规范

可空性限定符如上提到有 3 个版本（如 `nullable、_Nullable 、__nullable`）。首先弃用双下划线版本（如 `__nullable`）因为 Apple 保留它只是为了与 Xcode 6.3 兼容，搞不好以后哪个版本就去掉了。而 nullable 和 _Nullable 两个版本的区别在于放置位置以及修饰的指针类型。事实上，nullable 版本是 _Nullable 版本的简化形式，任何使用 nullable 版本的地方都可以使用 _Nullable 版本（除了 null_resettable），但优先使用 nullable 版本。对于属性以及方法中返回值和参数是简单对象或 Block 的指针类型，使用 nullable 版本；其它都使用 _Nullable 版本。

* 属性，使用 nullable 版本，作为 “属性关键字”。对于该放置开头还是末尾，我看 Apple 的库也没有统一的标准，不过我认为放开头更直观一点。

```objectivec
// Preferred
@property (nullable, nonatomic, copy) NSString *name;
// Not Preferred
@property (nonatomic, copy) NSString * _Nullable name;

// Generated Swift Interface
var name: String?
```

* 方法中，返回值和参数是简单对象或 Block 的指针类型，使用 nullable 版本，写在类型前面

```objectivec
// Preferred
- (nullable MyListItem *)itemWithName:(nonnull NSString *)name block:(nullable void (^)(void))block;
// Not Preferred
- (MyListItem * _Nullable)itemWithName:(NSString * _Nonnull)name block:(void (^ _Nullable)(void))block;
// 你可能会在 Apple 库中看到很多 block:(void (^ __nonnull)(void))block 的写法，那都是旧代码

// Generated Swift Interface
func item(withName name: String, block: (() -> Void)? = nil) -> MyListItem?
```

* 在 C 函数中，Block 的指针类型只能用 _Nullable 版本了

```objectivec
void block(void (^ _Nullable block)(void));
```

* 对于双指针、Block 的返回值、Block 的参数等其它所有支持的指针类型，不能用 nullable 版本修饰，只能使用 _Nullable 版本，写在类型后面

> 笔者注意到：Block 的返回值和参数类型，如果没有指定可空性的话，默认导入到 Swift 中是可选类型，而不是隐式解析可选类型。

```objectivec
- (void)param:(NSString * _Nullable * _Nullable)param;
- (void)param:(_Nullable id * _Nullable)param; // 也可以写成 (id _Nullable * _Nullable)
```

```objectivec
- (void)block:(nullable id _Nonnull (^)(id _Nullable params))block;

// Generated Swift Interface
func block(_ block: ((Any?) -> Any)? = nil)
```

#### Audited Regions：Nonnull 区域设置

在 Objective-C API 中，许多指针往往是 nonnull 的。而且如果每个属性或方法都去指定 nonnull 或 nullable，将是一件非常繁琐的事。Apple 为了减轻我们的工作量，提供了 audited Regions，也就是两个宏：`NS_ASSUME_NONNULL_BEGIN` 和 `NS_ASSUME_NONNULL_END`。在这两个宏之间的代码，所有未指定可空性限定符的简单指针类型都被假定为 nonnull，因此我们只需要去指定那些 nullable 指针类型即可。示例代码如下：

```objectivec
NS_ASSUME_NONNULL_BEGIN
@interface AAPLList : NSObject <NSCoding, NSCopying>
// ...
- (nullable AAPLListItem *)itemWithName:(NSString *)name;
- (NSInteger)indexOfItem:(AAPLListItem *)item;

@property (copy, nullable) NSString *name;
@property (copy, readonly) NSArray *allItems;
// ...
@end
NS_ASSUME_NONNULL_END

// --------------

self.list.name = nil;   // okay

AAPLListItem *matchingItem = [self.list itemWithName:nil];  // Warning: Null passed to a callee that requires a non-null argument
```

为了安全起见，此规则有一些例外：

* `typedef` 类型的可空性通常依赖于上下文，即使在 audited Regions 中也不能假定它为 nonnull。

我对该规则的理解是：以下 typedef 的 MyListBlock 类型，一般我们不去指定 MyListBlock 类型自身的可空性，而是让它依赖上下文。虽然指不指定都不影响 typedef 桥接到 Swift 中的结果，为 `typealias MyListBlock = (Any) -> Any?`。但是，如果你指定的话（比如指定为 nullable），那么使用到该类型地方就不能再指定为相反类型了（比如又指定为 nonnull），否则会报 nullability 类型冲突。

```objectivec
NS_ASSUME_NONNULL_BEGIN
typedef id _Nullable (^ _Nullable MyListBlock)(id);
- (void)param:(nonnull MyListBlock)block; // Error:  'nonnull' conflicts with existing specifier '_Nullable'
NS_ASSUME_NONNULL_END
```

所以我们不去指定 typedef 类型的可空性。而即使我们将它写在 audited Regions 中也不会假定它为 nonnull，所以使用到该类型的地方需要去指定可空性。比如以下 A 类声明中的两个方法中的参数类型的可空性，一个是由 audited Regions 假定为 nonnull，另一个是显式指定为 nullable 的。typedef MyListBlock 类型自身并没有携带可空性，即使它写在了 audited Regions 中。所以在 B 类声明中，由于没有使用 audited Regions，所以编译器给出了警告，说该 block 指针类型缺少可空性限定符，导入到 Swift 中时就成了隐式解析可选类型。

```objectivec
// C.h
NS_ASSUME_NONNULL_BEGIN
typedef id _Nullable (^ MyListBlock)(id);
NS_ASSUME_NONNULL_END
  
// A.h
NS_ASSUME_NONNULL_BEGIN
@interface A : NSObject
- (void)param:(MyListBlock)block; // 可空性由 audited Regions 假定为 nonnull
- (void)param1:(nullable MyListBlock)block; // 可空性显式指定为 nullable
@end
NS_ASSUME_NONNULL_END
  
// B.h
@interface B : NSObject
- (void)param:(MyListBlock)block; // Warning: block pointer is missing a nullability type specifier (_Nonnull, _Nullable, or _Null_unspecified)
@end
```

笔者还注意到一点， typedef Block 的参数类型的可空性可以被 audited Regions 假定为 nonnull。但返回值类型的可空性必须要显式指定，否则编译器会给出警告。前面提到过，Block 的返回值和参数类型，如果没有指定可空性的话，默认导入到 Swift 中是可选类型，而不是隐式解析可选类型。

```objectivec
// Objective-C interface
typedef id (^ MyListBlock0)(id); // Warning: pointer is missing a nullability type specifier (_Nonnull, _Nullable, or _Null_unspecified)
NS_ASSUME_NONNULL_BEGIN
typedef id (^ MyListBlock1)(id); // Warning: pointer is missing a nullability type specifier (_Nonnull, _Nullable, or _Null_unspecified)
typedef id _Nonnull (^ MyListBlock2)(id); // okay
NS_ASSUME_NONNULL_END
  
// Generated Swift Interface
typealias MyListBlock0 = (Any?) -> Any?
typealias MyListBlock1 = (Any) -> Any?
typealias MyListBlock2 = (Any) -> Any
```

* 对于复杂的指针类型（如 `id *、NSString **`）必须显式指定它的可空性。例如，要指定一个指向 nullable 对象引用的 nonnull 指针，可以使用 `_Nullable id * _Nonnull`；
* 特殊类型 `NSError **` 通常用于通过方法参数返回错误，Apple 总是假定它为是一个指向 nullable 的 NSError 对象引用的 nullable 的指针，所以可以不用显式指定。

为了保持一致性，Apple 建议我们在所有 Objective-C headers 中使用 audited Regions，同时这样也能保证混编的安全，即便你不去指定 nullable，因为导入到 Swift 中不再是隐式解析可选类型了。事实上，现在 Xcode 创建的 Objective-C header 默认都包含 audited Regions，你可以为所有旧的头文件中都添加 audited Regions 以先保证混编安全。同时，尽量避免使用 null_unspecified，除非将其作为过渡工具。

但是，仅仅是依靠 audited Regions 虽然解决了混编安全的问题，但这是不够的。因为 audited Regions 是假定区域内都为 nonnull 类型，这意味着导入到 Swift 中都是 non-optional 类型，没办法使用和传递 optional 类型或者 nil 了，Swift 类型安全会阻止你这么做。比如一个 nonnull 类型的 Objective-C 方法参数，你不能在 Swift 中将一个可选值或 nil 作为参数传给它 ；你也不能直接强制解析可选类型再传入，因为它可能为 nil，强制解析一个为 nil 的可选值会导致运行时错误。所以你需要添加一层保护，判断可选包含一个非 nil 的值再进行强制解析，或者使用可选绑定、 `??` 等手段。总之，一个原本在 Objective-C 中允许传入 nil 的参数，现在在 Swift 中不被允许了！因此，在 Objective-C 中你还需要指定那些 nullable 指针类型，才能更顺利地混编。

#### null_resettable

Objective-C APIs 还可以使用 null_resettable 来表示属性的 nullability（且只可用于属性），这些属性的 setter 允许设置 nil 以将值重置为某个默认值，但其 getter 永远不会返回 nil（因为提供了一个默认值）。

例如 UIView 的 tintColor 属性，当没有指定一个颜色时，它将使用一个默认的系统颜色。

```objectivec
@property (null_resettable, nonatomic, strong) UIColor *tintColor;
```

此类 API 导入到 Swift 中是 `隐式解析可选类型`。

```swift
var tintColor: UIColor!
```

使用 null_resettable 必须重写属性的 setter 或 getter 方法做非空处理。否则会报警告 `Synthesized setter 'setName:' for null_resettable property 'name' does not handle nil`，同时存在安全隐患。

笔者之前没有在代码中见过 null_resettable，也不明白它到底应该应用在什么场景。直到我遇到了一个需求，我希望我的 Optional String 属性 getter 要么返回一个值，要么返回 `""`，而不希望返回 nil，null_resettable 的作用与这相符。最后，我在 Swift 中写了一个属性包装器实现了它。

```swift
@propertyWrapper
struct NonnullString {
    private var string: String
    init() { string = "" }
    var wrappedValue: String! {
        get { return string }
        set { string = newValue ?? "" }
    }
}

// 泛型版本
protocol NonnullValueProtocol {
    init()
}

@propertyWrapper
struct NonnullValue<T: NonnullValueProtocol> {
    private var value: T
    init() { value = T() }
    var wrappedValue: T! {
        get { return value }
        set { value = newValue ?? T() }
    }
}

extension String: NonnullValueProtocol {}
extension Array: NonnullValueProtocol {}

class MyClass {
    @NonnullValue<String> var string: String!
    @NonnullValue<Array<String>> var array: Array<String>!
}
```

### 兼容性

向 Objective-C API 添加 nullability annotations 不会影响 Objective-C 代码向后兼容性或编译器生成代码的方式。例如，将 nil 赋值给 nonnull 指针也没有关系，只是在新的编译器下会给出警告，并不会报编译错误，也不会在运行时捕获到 nonnull 指针为 nil 而错误。为了向后兼容性，你仍然可以检查 nonnull 指针在运行时实际上是否为 nil。

你可以以使用断言或异常的方式来看待 nullability annotations，违反代码规范的约定就是程序员的错误。特别是，返回值是你可以控制的，因此如果返回值类型是 nonnull 的，你就不应该返回 nil，除非是为了向后兼容。

### 小结

如果没有正确使用 nullability annotations 为 Objective-C API 指定可空性，那么 Objective-C 声明中的指针类型导入到 Swift 中的类型将达不到预期，这给混编带来诸多不便，甚至存在安全隐患。nullability annotations 除了改善 Swift 中的体验外，还可以让开发者平滑地从 Objective-C 过渡到 Swift，而且让 Objective-C API 更具表现力且更易于正确使用，促使开发者在编写 Objective-C 代码时更加规范，减少同事之间的沟通成本。尽管它在 Xcode 6.3 就引入了，但大部分 Objective-C 开发者还是将它忽略，希望大家能重视起来，规范编码。

如有错误，欢迎指正！👏 🙏

### 参考

* [Apple｜Xcode Release Notes - Xcode 6](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW453)
* [Apple｜Xcode Release Notes - Xcode 6.3](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW378)
* [Apple｜Xcode Release Notes - Xcode 7](https://developer.apple.com/library/archive/releasenotes/DeveloperTools/RN-Xcode/Chapters/Introduction.html#//apple_ref/doc/uid/TP40001051-CH1-SW326)
* [Apple｜Nullability and Objective-C](https://developer.apple.com/swift/blog/?id=25)
* [Apple｜Designating Nullability in Objective-C APIs](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/designating_nullability_in_objective-c_apis)
