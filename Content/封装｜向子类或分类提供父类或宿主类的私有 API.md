## 向子类/分类提供父类/宿主类的私有 API

当在子类/分类中想要使用父类/宿主类的私有 API 时，我们不应该为了达成目的直接在父类的声明中 public 这些私有 API，这样破坏了类的封装性。可以参考 UIKit 中 UIGraphicsRenderer.h / UIGraphicsRendererSubclass.h、UIGestureRecognizer.h / UIGestureRecognizerSubclass.h 的做法。

比如 UIGestureRecognizer 类的声明如下：

```objectivec
// UIGestureRecognizer.h
@interface UIGestureRecognizer : NSObject
@property (nonatomic, readonly) UIGestureRecognizerState state;
// other public API
// ...
@end
```

对于想要在子类中使用的父类的私有 API，在 UIGestureRecognizerSubclass.h 中声明了个分类。我们知道分类的一个使用场合是创建对私有方法的前向引用，这里就是将父类的私有 API 公开。

```objectivec
// UIGestureRecognizerSubclass.h
#import <UIKit/UIGestureRecognizer.h>
// the extensions in this header are to be used only by subclasses of UIGestureRecognizer
// code that uses UIGestureRecognizers must never call these
@interface UIGestureRecognizer (UIGestureRecognizerProtected)
@property (nonatomic, readwrite) UIGestureRecognizerState state;  // the current state of the gesture recognizer. can only be set by subclasses of UIGestureRecognizer, but can be read by consumers
// other private API of UIGestureRecognizer
// ...
@end
```

目前为止这种方案还是存在缺点的。虽然在 UIGestureRecognizerSubclass.h 中已经注释分类声明中的 API 仅供子类使用，但如果使用方导入 UIGestureRecognizerSubclass.h 的话就可以访问 UIGestureRecognizer 类的私有 API 的，还是破坏了封装性，容易被误用。而且我们引入该头文件基本只会以 `#import <UIKit/UIKit.h>` 或 `@import UIKit;` 的形式，而不会使用 `#import <UIKit/UIGestureRecognizer.h>`，所以这个问题就一直存在。

对此，我们可以再改进一下，比如可以通过以下方案：

```objectivec
// 限制此头文件的使用
#ifdef UIGESTURERECOGNIZERSUBCLASS_PROTECTED_ACCESS
#import <UIKit/UIGestureRecognizer.h>
@interface UIGestureRecognizer (UIGestureRecognizerProtected)
// ...
@end
#else
#error Only be included by UIGestureRecognizerSubclass's subclass or category!
#endif
```

在子类/分类中需要使用父类/宿主类的私有 API 时：

```objectivec
// 开启使用权限
#define UIGESTURERECOGNIZERSUBCLASS_PROTECTED_ACCESS
// 导入 UIGestureRecognizerSubclass.h
#import <UIKit/UIGestureRecognizerSubclass.h>
```

如果在其它类中没有开启使用权限，意外导入 UIGestureRecognizerSubclass.h 的话就会编译错误。

