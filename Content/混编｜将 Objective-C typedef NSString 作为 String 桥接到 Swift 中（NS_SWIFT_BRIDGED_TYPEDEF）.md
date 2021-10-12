## 混编｜将 Objective-C typedef NSString 作为 String 桥接到 Swift 中（NS_SWIFT_BRIDGED_TYPEDEF）

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1634002390942-f364763e-db92-4079-80b1-46683722c37d.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

### 前言

在 Objective-C 与 Swift 混编的过程中，你是否遇到因为 Objective-C typedef 的桥接问题导致在 Swift 中使用时产生类型冲突的问题呢，本篇文章就带你使用 `NS_SWIFT_BRIDGED_TYPEDEF` 宏优雅地解决这一问题。

### NS_SWIFT_BRIDGED_TYPEDEF 使用场景

在 Objective-C 与 Swift 混编的过程中，我遇到了如下问题：

我在 Objective-C Interface 中使用 typedef 为 NSString * 取了一个有意义的类型别名 TimerID，但 Generated Swift Interface 却不尽人意。在方法参数中 TimerID 类型被转为了 String，而 TimerID 却还是 NSString 的类型别名。

```swift
// Objective-C Interface
typedef NSString * TimerID;

@interface Timer : NSObject
+ (void)cancelTimer:(TimerID)timerID NS_SWIFT_NAME(cancel(timerID:));

@end

// Generated Swift Interface
public typealias TimerID = NSString

open class Timer : NSObject {
     open class func cancel(timerID: String)
}
```

这在 Swift 中使用的时候就遇到了类型冲突问题。由于 TimerID 是 NSString 的类型别名，而 NSString 又不能隐式转换为 String。

```swift
// Use in Swift
let timerID: TimerID = ""
Timer.cancel(timerID: timerID) // Error: 'TimerID' (aka 'NSString') is not implicitly convertible to 'String'; did you mean to use 'as' to explicitly convert? Insert ' as String'
```

可以通过以下方式解决该问题：

1. 在 Swift 中放弃使用 TimerID 类型，全部用 String 类型
2. 在 Swift 中使用到 TimerID 的地方显示转化为 String 类型

```swift
Timer.cancel(timerID: timerID as String)
```

这不是我想要的结果，我思考着能不能从根源上解决该问题，也就是在 Generate Swift Interface 阶段将 `typedef NSString *TimerID` 转换为 `public typealias TimerID = String`。

在摸鱼周报小伙伴的帮助下，我找到了[解决方案](https://stackoverflow.com/questions/53219460/how-do-i-bridge-objc-typedef-nsstring-to-swift-as-string)：使用 `NS_SWIFT_BRIDGED_TYPEDEF` 宏将 Objective-C typedef NSString 作为 String 桥接到 Swift 中。

```swift
// Objective-C Interface
typedef NSString * TimerID NS_SWIFT_BRIDGED_TYPEDEF;

@interface Timer : NSObject
+ (void)cancelTimer:(TimerID)timerID NS_SWIFT_NAME(cancel(timerID:));

@end

// Generated Swift Interface
public typealias TimerID = String // change: NSString -> String

open class Timer : NSObject {
    open class func cancel(timerID: TimerID) // change:  String -> TimerID
}
```

现在，我可以在 Swift 中愉快地使用 TimerID 类型啦！

```swift
let timerID: TimerID = ""
Timer.cancel(timerID: timerID) 
```

除了 NSString，`NS_SWIFT_BRIDGED_TYPEDEF` 还可以用在 NSDate、NSArray 等其它 Objective-C 类型别名中。

### NS_SWIFT_BRIDGED_TYPEDEF vs NS_TYPED_ENUM

在 [混编｜为 Objective-C 添加枚举宏，改善混编体验](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E4%B8%BA%20Objective-C%20%E6%B7%BB%E5%8A%A0%E6%9E%9A%E4%B8%BE%E5%AE%8F%EF%BC%8C%E6%94%B9%E5%96%84%E6%B7%B7%E7%BC%96%E4%BD%93%E9%AA%8C.md) 文章中，我们在类似但不同的场景为 NSString 的 Objective-C typedef 添加了宏 `NS_TYPED_ENUM`，我们来看看将它放在此处会产生什么样的效果。

```swift
// Objective-C Interface
typedef NSString * TimerID NS_TYPED_ENUM;

@interface Timer : NSObject
+ (void)cancelTimer:(TimerID)timerID NS_SWIFT_NAME(cancel(timerID:));

@end

// Generated Swift Interface
public struct TimerID : Hashable, Equatable, RawRepresentable {
    public init(rawValue: String)
}

open class Timer : NSObject {
    open class func cancel(timerID: TimerID)
}

// Use in Swift
let timerID = TimerID(rawValue: "")
Timer.cancel(timerID: timerID)
```

TimerID 变成了 Struct，而不是 String 的类型别名。相比之下，显然使用 `NS_SWIFT_BRIDGED_TYPEDEF` 更简洁更方便。而且，不同宏的使用场景也不同，`NS_TYPED_ENUM` 宏是用于声明类型常量枚举，为了遵守代码规范不应该用在此处。

在 [Apple｜AppKit Release Notes for macOS 10.14 - New Macros in AppKit Headers](https://developer.apple.com/documentation/macos-release-notes/appkit-release-notes-for-macos-10_14) 中也提到，在 macOS 10.13 中，NSString 的 Objective-C typedef 是使用宏 `NS_EXTENSIBLE_STRING_ENUM` 来声明的，这在 Swift 中使用起来就不太简洁方便。

```swift
let nib = NSNib(nibNamed: NSNib.Name("Inspector"), bundle: nil)
```

而在 macOS 10.14 中 Apple 将它们改为了使用 `NS_SWIFT_BRIDGED_TYPEDEF` 来声明，以上 API 的调用就可以简化为：

```swift
let nib = NSNib(nibNamed: "Inspector", bundle: nil)
```

### One More Thing

在写这篇博客的时候，我在 Stack Overflow 上看到了一个很有意思的问题：为什么 `typedef NSString * MyString` 优于 `typedef NSString MyString`？链接在这 [Stack Overflow｜In objective-c, why 'typedef NSString * MyString' is preferred over 'typedef NSString MyString'?](https://stackoverflow.com/questions/45063451/in-objective-c-why-typedef-nsstring-mystring-is-preferred-over-typedef-nss)。

楼主认为 `typedef NSString * MyString` 隐藏了 MyString 是一个指针的事实，而且也没有以 `Ref` 后缀命名以指示其是一个指针，所以他更喜欢 `typedef NSString MyString`。然而 `typedef NSString MyString` 不能很好地与 Swift 配合使用。

于是笔者我尝试看看 `typedef NSString MyString` 在 Generated Swift Interface 中是什么样的。

```swift
// Objective-C Interface
typedef NSString MyString;
// Generated Swift Interface
// nothing
```

居然没有生成。不过确实好像很少看到 `typedef NSString MyString` 这样的写法，Apple 自己的代码写法也都是 `typedef NSString * MyString`，例如：

```swift
typedef NSString * NSErrorDomain;
typedef NSString * NSExceptionName NS_TYPED_EXTENSIBLE_ENUM;
typedef NSString * NSRunLoopMode NS_TYPED_EXTENSIBLE_ENUM;
```

在 Core Foundation 中定义的隐藏指针信息的类型别名是有以 `Ref` 后缀命名的，但在 Objective-C 中不遵循这样的命名。

```
typedef const struct CF_BRIDGED_TYPE(NSString) __CFString * CFStringRef;
typedef struct CF_BRIDGED_MUTABLE_TYPE(NSMutableString) __CFString * CFMutableStringRef;
```

### 小结

通过阅读本文，你是否掌握了 `NS_SWIFT_BRIDGED_TYPEDEF` 宏的使用了呢？如果你的工程也在使用 Objective-C 与 Swift 混编，不妨为 Objective-C typedef 加上 `NS_SWIFT_BRIDGED_TYPEDEF` 宏，以解决类型桥接问题。

### 参考

* [Apple｜AppKit Release Notes for macOS 10.14 - New Macros in AppKit Headers](https://developer.apple.com/documentation/macos-release-notes/appkit-release-notes-for-macos-10_14)
* [Stack Overflow｜How do I bridge objc typedef NSString to Swift as String?](https://stackoverflow.com/questions/53219460/how-do-i-bridge-objc-typedef-nsstring-to-swift-as-string)
* [Stack Overflow｜In objective-c, why 'typedef NSString * MyString' is preferred over 'typedef NSString MyString'?](https://stackoverflow.com/questions/45063451/in-objective-c-why-typedef-nsstring-mystring-is-preferred-over-typedef-nss)





