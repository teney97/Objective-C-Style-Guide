## 标记构造器为指定构造器

在 Objective-C 中，对象初始化是基于指定构造器的概念。指定构造器负责调用其父类的一个指定构造器，然后初始化自己的实例变量。非指定构造器的构造器成为便利构造器，便利构造器通常代理到另一个构造器，并最终在指定构造器处终止链，而不是自己执行初始化。

这里所说的构造器也就是 init 系列的任何方法；指定构造器（Designated initializer）通常就是接收全部初始化参数的全能构造器；便利构造器（Convenience (or Secondary) initializer）通常为接收部分初始化参数的构造器，它们调用当前类的其它构造器，并为一些参数赋默认值。

![](https://gitee.com/junteng/images/raw/master/img/20211227003007.png)

指定构造器模式有助于确保继承的构造器正确地初始化所有实例变量。需要指定构造器的子类应该覆写其父类的所有指定构造器，但它不需要覆写便利构造器。有关构造器的更多信息，参阅 [Apple｜Concepts in Objective-C Programming —— Object Initialization](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Initialization/Initialization.html#//apple_ref/doc/uid/TP40010810-CH6)。

为了明确区分指定构造器和便利构造器，你可以将 `NS_DESIGNATED_INITIALIZER` 宏添加到构造器中，将其表示为指定构造器，其它未添加该宏的构造器都为便利构造器。

```objectivec
- (instancetype)init NS_DESIGNATED_INITIALIZER;
```

使用这个宏会引入一些规则：

* 指定构造器只能向上代理到父类的一个指定构造器（with `[super init...]`）；
* 便利构造器（在至少有一个构造器被标记为指定构造器的类中，未标记为指定构造器的构造器都为便利构造器）的实现必须横向代理到另一个构造器（with `[self init...]`）。补充一点，最终需要在指定构造器处终止链；
* 如果一个类提供了一个或多个指定构造器，它必须覆写其父类的所有指定构造器，

如果违反了以上任何规则，将会得到编译器的警告。

简单来说，指定构造器必须总是*向上*代理，便利构造器必须总是*横向*代理。

![](https://gitee.com/junteng/images/raw/master/img/20211224150002.png)

在 Objective-C 中，使用宏 `NS_DESIGNATED_INITIALIZER` 标记构造器为指定构造器有如下优点：

* 它可以充分发挥编译器的特性帮我们找出初始化过程中可能存在的漏洞（通过警告），有助于确保继承的构造器正确地初始化所有实例变量，增强代码的健壮性。
* 便于使用者重写或 hook 指定构造器。例如 UIView 的指定构造器为 `- initWithFrame:` 而不是 `- init` ，因此我们在 UIView 的自定义子类中总是重写 `- initWithFrame:` 而不是 `- init`，当要 hook UIView 的构造器的时候也总是 hook  `- initWithFrame:`。

Demo：

```objectivec
@interface MyClass : NSObject
- (instancetype)initWithTitle:(nullable NSString *)title subTitle:(nullable NSString *)subTitle NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithTitle:(nullable NSString *)title;
- (instancetype)init;
@end
  
@implementation MyClass
  
- (instancetype)initWithTitle:(nullable NSString *)title subTitle:(nullable NSString *)subTitle {
    self = [super init]; // [规则1] 指定构造器只能向上代理到父类指定构造器，否则会得到编译器警告：Designated initializer should only invoke a designated initializer on 'super'
    if (self) {
        _title = title;
        _subTitle = subTitle;
    }
    return self;
}

- (instancetype)initWithTitle:(nullable NSString *)title {
/* 
    return [super init]; 
    [规则2] 当该类设定了指定构造器也就是使用了 NS_DESIGNATED_INITIALIZER 后，其它非指定构造器都变成了便利构造器。
    便利构造器只能横向代理到该类的指定构造器，或者通过横向代理到其它便利构造器最后间接代理到该类的指定构造器。
    这里调用 [super init] 的话会得到编译器警告：
    	- Convenience initializer missing a 'self' call to another initializer
    	- Convenience initializer should not invoke an initializer on 'super'
 */
    return [self initWithTitle:title subTitle:nil];
}

// [规则3] 如果子类提供了指定构造器，那么需要重写所有父类的指定构造器为子类的便利构造器，保证子类新增的实例变量能够被正确初始化，以让构造过程更完整。
// 这里需要重写 -init，否则会得到编译器警告：Method override for the designated initializer of the superclass '-init' not found
- (instancetype)init {
    return [self initWithTitle:nil subTitle:nil];
}

@end
```



### 参考

* [iOS: 聊聊 Designated Initializer（指定初始化函数）](http://www.cnblogs.com/smileEvday/p/designated_initializer.html)
* [Apple｜Adopting Modern Objective-C —— Object Initialization](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/ModernizationObjC/AdoptingModernObjective-C/AdoptingModernObjective-C.html#//apple_ref/doc/uid/TP40014150-CH1-SW8)
* [Apple｜Concepts in Objective-C Programming —— Object Initialization](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Initialization/Initialization.html#//apple_ref/doc/uid/TP40010810-CH6)
* [Apple｜Cocoa Core Competencies —— Multiple initializers](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/MultipleInitializers.html)
* [Apple｜Swift Initialization](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html)
* [SwiftGG｜Swift 教程 - 类的继承和构造过程](https://swiftgg.gitbook.io/swift/swift-jiao-cheng/14_initialization#class-inheritance-and-initialization)

