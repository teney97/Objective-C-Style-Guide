## 混编｜Objective-C alloc

在 Swift 中 alloc 方法被禁用。

```objectivec
// NSObject
+ (instancetype)alloc OBJC_SWIFT_UNAVAILABLE("use object initializers instead");
```

在 Swift 中实例化 Objective-C，也不会调用 alloc 方法，也是直接调用构造器。

这导致一些代码不兼容了，比如在基础库中定义了一个默认的 DefaultLoadingView。

```objectivec
@interface DefaultLoadingView : UIView <LoadingViewProtocol>
@end

@interface DefaultLoadingView (sub)
+ (Class)defaultLoadingViewClass4Sub;
@end

@implementation DefaultLoadingView
+ (instancetype)new {  
    return [self alloc];
}
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

如果在 Swift 中实例化 DefaultLoadingView 的话，将都是 DefaultLoadingView 类型。

解决方案：

1、通过协议工厂，同时兼容旧代码

```objectivec
@implementation DefaultLoadingView
+ (void)load {
    [ServiceFactory registerService:@protocol(LoadingViewProtocol) defaultImplClass:self.class];
}
@end
  
@implementation DefaultLoadingView (sub)
+ (void)load {
    [ServiceFactory registerService:@protocol(LoadingViewProtocol) implClass:CustomLoadingView.class];
}
@end
  
// 使用的时候通过协议工厂创建
let loadingView = ServiceFactory.createService(LoadingViewProtocol.self, initBlock: nil) as! UIView & LoadingViewProtocol
```

2、在 OC 中加个工厂方法

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
  
// 通过新的构造器方法
let loadingView = DefaultLoadingView.init(swift_frame: frame)
```

