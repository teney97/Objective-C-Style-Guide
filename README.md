# Objective-C-Style-Guide

## 混编

* [0.查看编译器为 Objective-C 接口生成的 Swift 接口](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E6%9F%A5%E7%9C%8B%E7%BC%96%E8%AF%91%E5%99%A8%E4%B8%BA%20Objective-C%20%E6%8E%A5%E5%8F%A3%E7%94%9F%E6%88%90%E7%9A%84%20Swift%20%E6%8E%A5%E5%8F%A3.md)
* [1.为 Objective-C 添加枚举宏，改善混编体验](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E4%B8%BA%20Objective-C%20%E6%B7%BB%E5%8A%A0%E6%9E%9A%E4%B8%BE%E5%AE%8F%EF%BC%8C%E6%94%B9%E5%96%84%E6%B7%B7%E7%BC%96%E4%BD%93%E9%AA%8C.md)
* [2.将 Objective-C typedef NSString 作为 String 桥接到 Swift 中（NS_SWIFT_BRIDGED_TYPEDEF）](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E4%BD%BF%E7%94%A8%20NS_SWIFT_BRIDGED_TYPEDEF%20%E5%B0%86%20Objective-C%20typedef%20NSString%20%E4%BD%9C%E4%B8%BA%20String%20%E6%A1%A5%E6%8E%A5%E5%88%B0%20Swift%20%E4%B8%AD.md)
* [3.为 Swift 重命名 Objective-C API（NS_SWIFT_NAME）](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/混编｜为%20Swift%20重命名%20Objective-C%20API（NS_SWIFT_NAME）.md)
* [4.为 Swift 改进 Objective-C API 声明（NS_REFINED_FOR_SWIFT）](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/混编｜为%20Swift%20改进%20Objective-C%20API%20声明（NS_REFINED_FOR_SWIFT）.md)
* [5.限制 API 可用性](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E9%99%90%E5%88%B6%20API%20%E5%8F%AF%E7%94%A8%E6%80%A7.md)
* [6.为 Objective-C API 指定可空性](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E4%B8%BA%20Objective-C%20API%20%E6%8C%87%E5%AE%9A%E5%8F%AF%E7%A9%BA%E6%80%A7.md)
* [7.Objective-C override alloc 的兼容性问题](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/混编｜Objective-C%20override%20alloc%20的兼容性问题.md)
* [8.CToSwiftNameTranslation](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9CCToSwiftNameTranslation.md)

概览

* `NS_ENUM`。用于声明简单枚举，将作为 `enum` 导入到 Swift 中。建议将使用其它方式来声明的 Objective-C 简单枚举进行改造，使用 `NS_ENUM` 来声明，以更好地在 Swift 中使用。
* `NS_CLOSED_ENUM`。用于声明不会变更枚举成员的简单枚举（简称 “冻结枚举” ），例如 NSComparisonResult，将作为 `@frozen enum` 导入到 Swift 中。冻结枚举以降低灵活性的代价，换取了性能上的提升。
* `NS_OPTIONS`。用于声明选项枚举，将作为 `struct` 导入到 Swift 中。
* `NS_TYPED_ENUM`。用于声明类型常量枚举，将作为 `struct` 导入到 Swift 中。可大大改善 Objective-C 类型常量在 Swift 中的使用方式。
* `NS_TYPED_EXTENSIBLE_ENUM`。用于声明可扩展的类型常量枚举。与 `NS_TYPED_ENUM` 的区别是生成的 `struct` 多了一个忽略参数标签的构造器。
* `NS_STRING_ENUM` / `NS_EXTENSIBLE_STRING_ENUM`。用于声明字符串常量枚举，建议弃用，使用 `NS_TYPED_ENUM` / `NS_TYPED_EXTENSIBLE_ENUM` 替代。在 Xcode 13 中，Apple 已经将原先使用 `NS_EXTENSIBLE_STRING_ENUM` 声明的 NSNotificationName 等常量类型改为使用 `NS_TYPED_EXTENSIBLE_ENUM` 来声明。
* `NS_SWIFT_BRIDGED_TYPEDEF`。将 Objective-C typedef NSString 作为 String 桥接到 Swift 中，同样也支持其它类型，解决类型冲突问题。
* `NS_SWIFT_NAME`。用于在混编时为 Swift 重命名 Objective-C API，它可用在类、协议、枚举、属性、方法或函数、类型别名等等之中，让 Objective-C API 在 Swift 中有更合适的名称。相反，可以使用 `@objc(name)` 为 Objective-C 重命名 Swift API。
* `NS_REFINED_FOR_SWIFT`。用来为 Swift 改进 Objective-C API。具体是，在 Swift 中隐藏 Objective-C API，以便在 Swift 中提供相同 API 的更好版本，同时仍然可以使用原始 Objective-C 实现。以此来改进 Objective-C API。
* `NS_SWIFT_UNAVAILABLE` ，使 Objective-C API 在 Swift 中不可用； `NS_UNAVAILABLE` ，使 Objective-C API 在 Objective-C 和 Swift 中都不可用。
* `nullability annotations`。为 Objective-C API 指定可空性，以控制 Objective-C 声明中的指针类型如何导入到 Swift 中，让混编更安全更顺利；而且可以让开发者平滑地从 Objective-C 过渡到 Swift；还让 Objective-C API 更具表现力且更易于正确使用，促使开发者在编写 Objective-C 代码时更加规范，减少同事之间的沟通成本。


## 其它

基本规范
* [1.协议命名](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/基本规范｜协议命名.md)

语法
* [1.多用字面量语法，少用与之等价的方法](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E8%AF%AD%E6%B3%95%EF%BD%9C%E5%A4%9A%E7%94%A8%E5%AD%97%E9%9D%A2%E9%87%8F%E8%AF%AD%E6%B3%95%EF%BC%8C%E5%B0%91%E7%94%A8%E4%B8%8E%E4%B9%8B%E7%AD%89%E4%BB%B7%E7%9A%84%E6%96%B9%E6%B3%95%20.md)
* [2.多用类型常量，少用 #define 预处理指令](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/语法｜多用类型常量，少用%20%23define%20预处理指令.md)

if 语句
* [1.使用 Yoda Condition，将常量前置，以避免手误将 == 写成 =](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/if%20语句｜Yoda%20Condition.md)
* [2.使用布尔表达式，而不是隐式地与 0 做对比](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/if%20语句｜使用布尔表达式，而不是隐式地与%200%20做对比.md)
* [3.复杂条件处理1 - 提高可调式性](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/if%20语句｜复杂条件处理1%20-%20提高可调式性.md)
* [4.复杂条件处理2 - 前置计算复杂度低的子条件](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/if%20语句｜复杂条件处理2%20-%20前置计算复杂度低的子条件.md)
* [5.使用卫语句提前退出](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/if%20语句｜使用卫语句提前退出.md)

枚举
* [1.枚举在 switch 里的用法规范](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/枚举｜枚举在%20switch%20里的用法规范.md)

头文件
* [1.在类的头文件中尽量少引入其他头文件](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/头文件｜在类的头文件中尽量少引入其他头文件.md)
* [2.头文件引入整理](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/头文件｜头文件引入整理.md)
* [3.头文件｜向子类或分类提供父类（宿主类）的私有 API](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/头文件｜向子类或分类提供父类（宿主类）的私有%20API.md)

Block
* [1.避免在 block 内部直接使用成员变量](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/Block｜避免在%20block%20内部直接使用成员变量.md)
* [2.为常用的 block 类型创建 typedef](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/Block｜为常用的%20block%20类型创建%20typedef.md)

运算符
* [1.使用三目运算符时将默认值后置](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/运算符｜使用三目运算符时将默认值后置.md)

