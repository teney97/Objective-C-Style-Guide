## 标记构造器为指定构造器

关于指定构造器和便利构造器，可以阅读 [SwiftGG｜Swift 教程 - 类的继承和构造过程](https://swiftgg.gitbook.io/swift/swift-jiao-cheng/14_initialization#class-inheritance-and-initialization)。

指定构造器必须总是*向上*代理，便利构造器必须总是*横向*代理。

![](https://gitee.com/junteng/images/raw/master/img/20211224150002.png)

在 Objective-C 中，可以使用宏 `NS_DESIGNATED_INITIALIZER` 标记构造器为指定构造器。

* 它可以充分发挥编译器的特性帮我们找出初始化过程中可能存在的漏洞（通过警告），增强代码的健壮性。
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
    self = [super init]; // 指定构造器只能向上代理到父类指定构造器，否则会得到编译器警告：Designated initializer should only invoke a designated initializer on 'super'
    if (self) {
        _title = title;
        _subTitle = subTitle;
    }
    return self;
}

- (instancetype)initWithTitle:(nullable NSString *)title {
/* 
    return [super init]; 
    当该类设定了指定构造器也就是使用了 NS_DESIGNATED_INITIALIZER 后，其它构造器都变成了便利构造器。
    便利构造器只能横向代理到该类的指定构造，或者通过横向代理到其它便利构造器最后间接代理到该类的指定构造器。
    这里调用 [super init] 的话会得到编译器警告：
    	- Convenience initializer missing a 'self' call to another initializer
    	- Convenience initializer should not invoke an initializer on 'super'
 */
    return [self initWithTitle:title subTitle:nil];
}

// 如果子类提供了指定初始化函数，那么需要重写所有父类的指定构造器为子类的便利构造器，保证子类新增的变量能够被正确初始化，以让构造过程更完整。
// 这里需要重写 -init，否则会得到编译器警告：Method override for the designated initializer of the superclass '-init' not found
- (instancetype)init {
    return [self initWithTitle:nil subTitle:nil];
}

@end
```

### 参考

* [iOS: 聊聊 Designated Initializer（指定初始化函数）](http://www.cnblogs.com/smileEvday/p/designated_initializer.html)
* [Apple｜Swift Initialization](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html)
* [Apple｜Objective-C Initialization](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/ModernizationObjC/AdoptingModernObjective-C/AdoptingModernObjective-C.html#//apple_ref/doc/uid/TP40014150-CH1-SW8)
* [Apple｜Multiple initializers](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/MultipleInitializers.html)

