

## Object Initialization

初始化将对象的实例变量设置为合理且有用的初始值。它还可以分配和准备对象所需的其他全局资源，必要时从外部资源（如文件）加载它们。每个声明实例变量的对象都应该实现一个初始化方法 —— 除非默认的 set-everything-to-zero 初始化已经足够。如果一个对象没有实现初始化器，Cocoa 会调用最近的祖先的初始化器。

### 构造器的形式

NSObject 为构造器声明了 init 原型，它是一个返回 id（现在使用 instanceType）类型对象的实例方法。重写 init 对于不需要额外数据来初始化对象的子类来说是很好的。但是通常初始化依赖于外部数据来将对象设置为合理的初始状态。例如，假设您有一个 Account 类，正确初始化 Account 对象需要一个唯一的 account number 帐号，并且这个帐号必须提供给构造器。因此，构造器可以接受一个或多个参数，唯一的要求是初始化方法以字母 “init” 开头。（风格约定 init… 有时用来指构造器。)

**注意**：子类除了实现一个带参数的构造器，还可以实现简单的 init 构造器，然后在初始化后立即使用 setter 访问器方法将对象设置为有用的初始状态。或者，如果子类使用属性，它可以在初始化后立即给属性赋值。

Cocoa 有很多带参数的构造器示例：

```objectivec
- (id)initWithArray:(NSArray *)array; (from NSSet)
- (id)initWithTimeInterval:(NSTimeInterval)secsToBeAdded sinceDate:(NSDate *)anotherDate; (from NSDate)
- (id)initWithContentRect:(NSRect)contentRect styleMask:(unsigned int)aStyle backing:(id)bufferingType defer:(BOOL)flag; (from NSWindow)
- (id)initWithFrame:(NSRect)frameRect; (from NSControl and NSView)
```

这些构造器是以 “init” 开头的实例方法，并返回一个动态类型 id 对象（现在使用 instanceType）。除此之外，它们遵循 Cocoa 对多参数方法的约定，通常在第一个也是最重要的参数之前使用 `With` Type: 或 `From` Source: 。

### 构造器问题

虽然 init... 方法的方法签名要求返回一个对象，它不一定是最近分配的对象 —— 也就是 init... 消息的接收者。换句话说，，你从构造器获得的对象可能不是你认为的正在被初始化的对象。

有两个条件提示返回除了刚刚分配的对象之外的其他对象。第一种情况涉及到两种相关的情况：当必须有一个单例实例时，或者当对象的定义属性必须是唯一的时。一些 Cocoa 类 —— 例如 NSWorkspace —— 在一个程序中只允许一个实例。在这种情况下，类必须确保（在构造中，或者更可能的是在类工厂方法中）只创建一个实例，如果有任何对新实例的进一步请求，则返回这个实例。

当一个对象需要具有使唯一的属性时，也会出现类似的情况。回想一下前面提到的假设的 Account 类。任何类型的帐户都必须具有唯一标识符。如果这个类的初始化器（比如 initWithAccountID: ) 传递了一个已经与对象关联的标识符，它必须做两件事:

* 释放新分配的对象（在内存管理的代码中）
* 返回先前使用此唯一标识符初始化的 Account 对象

通过这样做，构造器确保标识符的唯一性，同时提供被请求的内容：带有被请求标识符的 Account 实例。

有时候一个 init... 方法无法执行所请求的初始化。例如，initFromFile: 方法期望从文件的内容中初始化一个对象，文件的路径作为参数传递。但如果该位置不存在文件，则无法初始化该对象。如果传递给 initWithArray: 构造器的是 NSDictionary 对象而不是 NSArray 对象，也会发生类似的问题。当一个 init... 构造器不能初始化对象，它应该：

* 释放新分配的对象（在内存管理的代码中）
* 返回 nil

从构造器返回 nil 表示不能创建请求的对象。当你创建一个对象时，你通常应该在继续之前检查返回值是否为 nil :

```objectivec
id anObject = [[MyClass alloc] init];
if (anObject) {
    [anObject doSomething];
    // more messages...
} else {
    // handle error
}
```

因为一个 init... 方法可能返回 nil 或其他对象，使用由 alloc 或 allocWithZone 返回的实例而不是由构造器返回的实例是危险的。考虑以下代码:

```objectivec
id myObject = [MyClass alloc];
[myObject init];
[myObject doSomething];
```

这个例子中的 init 方法可以返回 nil，也可以替换一个不同的对象。因为您可以向 nil 发送消息而不引发异常，所以在前一种情况下，除了（可能）调试方面的麻烦之外，什么也不会发生。但是，你应该始终依赖于已初始化的实例，而不是刚刚分配的“原始”实例。因此，你应该将 alloc 消息嵌套在 init 消息中（如 [][MyClass alloc] init]），并在继续之前测试从构造器返回的对象。

```objectivec
id myObject = [[MyClass alloc] init];
if (myObject) {
    [myObject doSomething];
} else {
    // error recovery...
}
```

一旦一个对象被初始化，你就不应该再初始化它。如果尝试重新初始化，实例化对象的框架类通常会引发异常。例如，本例中的第二次初始化将导致引发 NSInvalidArgumentException 异常。

```objectivec
NSString *aStr = [[NSString alloc] initWithString:@"Foo"];
aStr = [aStr initWithString:@"Bar"];
```

### 实现构造器

在实现 init... 方法时，有几个关键的规则要遵循。如果一个构造器是该类中的唯一构造器，或者作为类中多个构造器中的指定构造器，需要遵循：

* 总是首先调用父类的构造器
* 检查由父类构造器返回的对象，如果为 nil，则不能进行初始化，返回 nil 给消息接收者
* 当初始化引用对象的实例变量时，根据需要 retain 或 copy 对象
* 将实例变量设置为有效的初始值后，返回 self。除非：
  * 必须返回一个替换的对象，在这种情况下，首先 release 刚刚分配的对象
  * 一个问题阻止了初始化的成功，在这种情况下，返回 nil

```objectivec
- (id)initWithAccountID:(NSString *)identifier {
    if (self = [super init]) {
        Account *ac = [accountDictionary objectForKey:identifier];
        if (ac) { // object with that ID already exists
            [self release];
            return [ac retain];
        }
        if (identifier) {
            accountID = [identifier copy]; // accountID is instance variable
            [accountDictionary setObject:self forKey:identifier];
            return self;
        } else {
            [self release];
            return nil;
        }
    } else
        return nil;
}
```

**注意**：为了简单起见，如果参数为 nil，这个示例将返回 nil，但 Cocoa 的更好做法是引发异常。

在 ARC 下，以上实例代码可优化为：

```objectivec
- (instanceType)initWithAccountID:(NSString *)identifier {
    if (identifier == nil) {
        NSAssert(NO, @"identifier can not be nil");
        return nil; 
    } 
    self = [super init];
    if (self) {
        Account *ac = [accountDictionary objectForKey:identifier];
        if (ac) { return ac; }
        accountID = [identifier copy];
        [accountDictionary setObject:self forKey:identifier];
        return self;
    }
    return self;
}
```

没有必要显式初始化对象的所有实例变量，只需要初始化那些使对象功能正常所必需的实例变量即可。在分配过程中对实例变量执行的默认置零初始化（ The default set-to-zero initialization，比如 Int 类型的值）通常就足够了。根据内存管理的需要，确保 retain 或 copy 实例变量。

要求调用父类构造器作为第一步是很重要的。回想一下，一个对象不仅封装了它的类所定义的实例变量。而且封装了它的所有父类所定义的实例变量。通过首先调用父类的构造器，可以确保继承链上的类定义的实例变量首先被初始化。正确的初始化顺序至关重要，因为子类的后期初始化可能依赖于父类定义的实例变量被初始化为合理的值。

![](https://gitee.com/junteng/images/raw/master/img/20211230110357.png)

当你创建一个子类时，继承的构造器是一个问题。有时父类的 init... 方法足够初始化类的实例。但因为它很可能不会，所以你应该重写父类的构造器。如果不这样做，就会调用父类的实现，而且因为父类对子类一无所知，所以可能无法正确初始化实例。解决方案可以详见以下指定构造器章节。

### 多个构造器和指定构造器

一个类可以定义多个构造器。有时多个构造器让类的使用者以不同的形式为相同的初始化提供输入。例如，NSSet 类为使用者提供了几个接收不同形式的相同数据的构造器，如下：

```objectivec
- (id)initWithArray:(NSArray *)array;
- (id)initWithObjects:(id *)objects count:(unsigned)count;
- (id)initWithObjects:(id)firstObj, ...;
```

一些子类提供了便利构造器，为接收全部初始化参数的构造器提供默认值。这个接收全部初始化参数的构造器通常是指定构造器，是类中最重要的构造器。例如，假设有一个 Task 类，它有一个指定构造器：

```objectivec
- (id)initWithTitle:(NSString *)aTitle date:(NSDate *)aDate;
```

Task 类可能包括便利构造器，这些构造器只是调用指定构造器，并为便利构造器没有显式请求的参数传递默认值。以下是 Task 类的便利构造器：

```objectivec
- (id)initWithTitle:(NSString *)aTitle {
    return [self initWithTitle:aTitle date:[NSDate date]];
}
 
- (id)init {
    return [self initWithTitle:@"Task"];
}
```

指定构造器对类起着重要的作用。它确保通过调用父类的指定构造器来初始化继承的实例变量。它通常是有最多参数的 init… 方法，并且做了大部分的初始化工作，它是类的便利构造器通过横向代理最终代理到的构造器。

当你定义一个子类时，你必须能够识别父类的指定构造器，并在子类的指定构造器中调用它。你还必须确保以某种方式覆写继承的构造器。你可以根据需要提供尽可能多的便利构造器。在设计类的构造器时，要记住指定构造器是向上代理到父类的指定构造器的（with `[super init...]`）； 而其他的构造器，也就是便利构造器，则通过横向代理到该类的其他构造器，并最终在指定构造器处终止链（with `[self init...]`）。

举个例子会更清楚。假设有三个类 A、 B、C，类 B 继承了类 A，类 C 继承了类 B。每个子类都添加了一个属性作为实例变量，并实现了一个 init… 方法（指定构造器）来初始化该实例变量。它们还定义了便利构造器，并确保在必要时重写继承的构造器。下图说明了这三个类的构造器及其关系。

![](https://gitee.com/junteng/images/raw/master/img/20211230135248.png)

每个类的指定构造器是覆盖范围最大的构造器。指定构造器是初始化由子类添加的属性的构造器，也是负责调用父类指定构造器的构造器。在这里例子中，类 C 的指定构造器 `initWithTitle:date:` 调用它的父类 B 的指定构造器 `initWithTitle:`，而 `initWithTitle:` 又调用父类 A 的指定构造器 `init` 。在创建子类时，知道父类的指定构造器总是很重要的。

指定构造器总是向上代理，而便利构造器总是横向代理。便利构造器（如本例中所示）通常是继承的构造器的覆写版本。类 C 覆写 `initWithTitle:` 调用它的指定构造器，传递给它一个默认 date。这个指定构造器，调用类 B 的指定构造器，也就是被覆写的方法 `initWithTitle:`。如果你发送一个 `initWithTitle:` 消息给类 B 和类 C 的对象，你将调用不同的方法实现。另一方面，如果类 C 没有覆写 `initWithTitle:`，并且你将消息发送到类 C 的实例，类 B 实现将被调用。因此，C 实例将不完全初始化（因为它将缺少 date）。当创建一个子类时，确保所有继承的指定构造器都被充分覆写是很重要的。

有时，父类的指定构造器对子类来说可能已经足够了，因此子类不需要实现自己的指定构造器。其他时候，类的指定构造器可能是其父类的指定构造器的覆写版本，这种情况通常发生在：子类需要补充父类的指定构造器所执行的工作时，即使子类没有添加自己的任何实例变量（或者它添加的实例变量不需要显式初始化）。

## 标记构造器为指定构造器

在 Objective-C 中，对象初始化是基于指定构造器的概念。指定构造器负责调用其父类的一个指定构造器，然后初始化自己的实例变量。非指定构造器的构造器是便利构造器，便利构造器通常代理到另一个构造器，并最终在指定构造器处终止链，而不是自己执行初始化。

这里所说的构造器也就是 init 系列的任何方法；指定构造器（Designated initializer）通常就是接收全部初始化参数的全能构造器；便利构造器（Convenience (or Secondary) initializer）通常为接收部分初始化参数的构造器，它们调用当前类的其它构造器，并为一些参数赋默认值。

![](https://gitee.com/junteng/images/raw/master/img/20211227003007.png)

指定构造器模式有助于确保继承的构造器正确地初始化所有实例变量。需要指定构造器的子类应该覆写其父类的所有指定构造器，但它不需要覆写便利构造器。有关构造器的更多信息，参阅 [Apple｜Concepts in Objective-C Programming —— Object Initialization](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Initialization/Initialization.html#//apple_ref/doc/uid/TP40010810-CH6)。

为了明确区分指定构造器和便利构造器，你可以将 `NS_DESIGNATED_INITIALIZER` 宏添加到构造器中，将其表示为指定构造器，其它未添加该宏的构造器都为便利构造器。

```objectivec
- (instancetype)init NS_DESIGNATED_INITIALIZER;
```

使用这个宏会引入一些规则：

* 指定构造器只能且必须向上代理到父类的一个指定构造器（with `[super init...]`）；
* 便利构造器（在至少有一个构造器被标记为指定构造器的类中，未标记为指定构造器的构造器都为便利构造器）的实现只能且必须横向代理到另一个构造器（with `[self init...]`），最终需要在指定构造器处终止链；
* 如果一个类提供了一个或多个指定构造器，它必须覆写其父类的所有指定构造器作为（退化为）该类的便利构造器，并让其满足条件 2。这样才能保证子类新增的实例变量得到正确的初始化。

如果违反了以上任何规则，将会得到编译器的警告。

简单来说，指定构造器必须总是*向上*代理，便利构造器必须总是*横向*代理。

![](https://gitee.com/junteng/images/raw/master/img/20211224150002.png)

在 Objective-C 中，使用宏 `NS_DESIGNATED_INITIALIZER` 标记构造器为指定构造器有如下优点：

* 它可以充分发挥编译器的特性帮我们找出初始化过程中可能存在的漏洞（通过警告），有助于确保继承的构造器正确地初始化所有实例变量，增强代码的健壮性。
* 便于使用者重写或 hook 指定构造器。例如 UIView 的指定构造器为 `- initWithFrame:` 而不是 `- init` ，因此我们在 UIView 的自定义子类中总是重写 `- initWithFrame:` 而不是 `- init`，当要 hook UIView 的构造器的时候也总是 hook  `- initWithFrame:`。

Demo：

```objectivec
@interface MyClass : NSObject
- (instancetype)initWithTitle:(nullable NSString *)title subtitle:(nullable NSString *)subtitle NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithTitle:(nullable NSString *)title;
- (instancetype)init;
@end
  
@implementation MyClass
  
- (instancetype)initWithTitle:(nullable NSString *)title subtitle:(nullable NSString *)subtitle {
    self = [super init]; // [规则1] 指定构造器只能向上代理到父类指定构造器，否则会得到编译器警告：Designated initializer should only invoke a designated initializer on 'super'
    if (self) {
        _title = [title copy];
        _subtitle = [subtitle copy];
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
    return [self initWithTitle:title subtitle:nil];
}

// [规则3] 如果子类提供了指定构造器，那么需要重写所有父类的指定构造器为子类的便利构造器，保证子类新增的实例变量能够被正确初始化，以让构造过程更完整。
// 这里需要重写 -init，否则会得到编译器警告：Method override for the designated initializer of the superclass '-init' not found
- (instancetype)init {
    return [self initWithTitle:nil];
}

@end
```



### 参考

* [iOS: 聊聊 Designated Initializer（指定初始化函数）](http://www.cnblogs.com/smileEvday/p/designated_initializer.html)
* [Apple｜Cocoa Core Competencies —— Object creation](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/ObjectCreation.html#//apple_ref/doc/uid/TP40008195-CH39-SW1)
* [Apple｜Cocoa Core Competencies —— Initialization](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Initialization.html#//apple_ref/doc/uid/TP40008195-CH21-SW1)
* [Apple｜Adopting Modern Objective-C —— Object Initialization](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/ModernizationObjC/AdoptingModernObjective-C/AdoptingModernObjective-C.html#//apple_ref/doc/uid/TP40014150-CH1-SW8)
* [Apple｜Concepts in Objective-C Programming —— Object Initialization](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Initialization/Initialization.html#//apple_ref/doc/uid/TP40010810-CH6)
* [Apple｜Cocoa Core Competencies —— Multiple initializers](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/MultipleInitializers.html)
* [Apple｜Swift Initialization](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html)
* [SwiftGG｜Swift 教程 - 类的继承和构造过程](https://swiftgg.gitbook.io/swift/swift-jiao-cheng/14_initialization#class-inheritance-and-initialization)

