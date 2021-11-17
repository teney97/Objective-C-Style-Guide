## 混编｜Objective-C override alloc 的兼容性问题

今天在混编时遇到了 Objective-C override alloc 的兼容性问题。

### 问题

在 Swift 中 alloc 方法被禁用了，而且在 Swift 中实例化 Objective-C，也不会调用 alloc 方法，也是直接调用构造器方法。

```objectivec
+ (instancetype)alloc OBJC_SWIFT_UNAVAILABLE("use object initializers instead");
```

这导致一些代码不兼容了，比如在基础库中定义了一个 DefaultLoadingView。

```objectivec
@interface DefaultLoadingView : UIView <LoadingViewProtocol>
@end

@interface DefaultLoadingView (sub)
+ (Class)defaultLoadingViewClass4Sub;
@end

@implementation DefaultLoadingView
+ (instancetype)alloc {
    if ([self respondsToSelector:@selector(defaultLoadingViewClass4Sub)]) {
        Class tempClass = [self defaultLoadingViewClass4Sub];
        if (tempClass) {
            return [tempClass alloc];
        }
    }
    return [super alloc];
}
@end
```

业务方可以通过实现 DefaultLoadingView+sub 分类来自定义 loadingView。

```objectivec
@implementation DefaultLoadingView (sub)
+ (Class)defaultLoadingViewClass4Sub {
    return CustomLoadingView.class;
}
@end
```

在 Objective-C 我们可以直接通过 [[DefaultLoadingView alloc] init] 来获取 loadingView 实例，如果业务方有自定义的那获取到的就是 CustomLoadingView 类型，否则就是 DefaultLoadingView 类型。

但在 Swift 中通过 DefaultLoadingView() 获取实例的话，都将是 DefaultLoadingView 类型，因为它不会调用 alloc 方法。

### 解决方案

1、如果要继续保留采用 alloc 方案的话，就在 Objective-C 中包个中间层。比如这里我们可以新增一个工厂方法来调用 alloc 和构造器方法，然后使用 `NS_SWIFT_NAME` 重命名该工厂方法使其作为构造器导入到 Swift 中以方便使用，同时使用 `NS_SWIFT_UNAVAILABLE` 将原先的构造器方法标记标记为在 Swift 中不可用且指示调用者使用新的构造器方法。

```objectivec
@interface DefaultLoadingView : UIView <LoadingViewProtocol>
+ (instancetype)loadingViewWithFrame:(CGRect)frame NS_SWIFT_NAME(init(swift_frame:));
- (instancetype)initWithFrame:(CGRect)frame NS_SWIFT_UNAVAILABLE("use init(swift_frame:)");
@end
  
@implementation DefaultLoadingView
+ (instancetype)loadingViewWithFrame:(CGRect)frame {
    return [[self alloc] initWithFrame:frame];
}
@end
  
// 使用新的构造器方法
let loadingView = DefaultLoadingView.init(swift_frame: frame)
```

这个方案的缺点是：如果原先构造器方法有多个，那么就要新增多个对应的工厂方法，并添加 `NS_SWIFT_NAME` 和 `NS_SWIFT_UNAVAILABLE`。

2、针对以上这个场景，我们可以使用另外一种方案 —— 在 Swift 中直接取出想要的那个类，摒弃 alloc 方案，当然你也可以为了向后兼容对其进行保留。通过协议工厂（维护 protocol 和 implClass 的映射关系），直接取出想要的那个类。在基础库中对协议 LoadingViewProtocol 注册 defaultImplClass DefaultLoadingView，业务方可以对协议注册 implClass CustomLoadingView。使用的时候通过协议去工厂中取出 implClass 或 implInstance，如果未注册 implClass 则取默认的 defaultImplClass 或 defaultImplInstance。

```objectivec
@implementation DefaultLoadingView
+ (void)load {
    [ServiceFactory registerService:@protocol(LoadingViewProtocol) defaultImplClass:self.class];
}
@end
  
@implementation CustomLoadingView
+ (void)load {
    [ServiceFactory registerService:@protocol(LoadingViewProtocol) implClass:CustomLoadingView.class];
}
@end
  
// 使用的时候通过协议工厂创建
let loadingView = ServiceFactory.createService(LoadingViewProtocol.self, initBlock: nil) as! UIView & LoadingViewProtocol
```

不知道你有没有更好的方案呢？欢迎留言。
