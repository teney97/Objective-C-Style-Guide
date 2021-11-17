# Objective-C-Style-Guide

> 争取一周两三更吧。

## 目录

混编：
* [0.查看编译器为 Objective-C 接口生成的 Swift 接口](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E6%9F%A5%E7%9C%8B%E7%BC%96%E8%AF%91%E5%99%A8%E4%B8%BA%20Objective-C%20%E6%8E%A5%E5%8F%A3%E7%94%9F%E6%88%90%E7%9A%84%20Swift%20%E6%8E%A5%E5%8F%A3.md)
* [1.为 Objective-C 添加枚举宏，改善混编体验](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E4%B8%BA%20Objective-C%20%E6%B7%BB%E5%8A%A0%E6%9E%9A%E4%B8%BE%E5%AE%8F%EF%BC%8C%E6%94%B9%E5%96%84%E6%B7%B7%E7%BC%96%E4%BD%93%E9%AA%8C.md)
* [2.将 Objective-C typedef NSString 作为 String 桥接到 Swift 中（NS_SWIFT_BRIDGED_TYPEDEF）](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E4%BD%BF%E7%94%A8%20NS_SWIFT_BRIDGED_TYPEDEF%20%E5%B0%86%20Objective-C%20typedef%20NSString%20%E4%BD%9C%E4%B8%BA%20String%20%E6%A1%A5%E6%8E%A5%E5%88%B0%20Swift%20%E4%B8%AD.md)
* [3.为 Swift 重命名 Objective-C API（NS_SWIFT_NAME）](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/混编｜为%20Swift%20重命名%20Objective-C%20API（NS_SWIFT_NAME）.md)
* [4.为 Swift 改进 Objective-C API 声明（NS_REFINED_FOR_SWIFT）](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/混编｜为%20Swift%20改进%20Objective-C%20API%20声明（NS_REFINED_FOR_SWIFT）.md)
* [5.限制 API 可用性](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E9%99%90%E5%88%B6%20API%20%E5%8F%AF%E7%94%A8%E6%80%A7.md)
* [6.为 Objective-C API 指定可空性](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9C%E4%B8%BA%20Objective-C%20API%20%E6%8C%87%E5%AE%9A%E5%8F%AF%E7%A9%BA%E6%80%A7.md)
* [7.Objective-C override alloc 的兼容性问题](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/混编｜Objective-C%20override%20alloc%20的兼容性问题.md)
* [8.CToSwiftNameTranslation](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9CCToSwiftNameTranslation.md)

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

头文件引入
* [1.在类的头文件中尽量少引入其他头文件](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/头文件引入｜在类的头文件中尽量少引入其他头文件.md)
* [2.头文件引入整理](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/头文件引入｜头文件引入整理.md)

Block
* [1.避免在 block 内部直接使用成员变量](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/Block｜避免在%20block%20内部直接使用成员变量.md)
* [2.为常用的 block 类型创建 typedef](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/Block｜为常用的%20block%20类型创建%20typedef.md)

运算符
* [1.使用三目运算符时将默认值后置](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/运算符｜使用三目运算符时将默认值后置.md)


属性
* 1.在对象内部到底该通过属性还是成员变量来读写数据？
* 2.在设置属性所对应的成员变量时，一定要遵从该属性所声明的语义
* 3.weak 和 unsafe_unretained 如何选择？
* 4.为了达到封装的目的，我们应该只在确有必要时才将属性对外暴露，并且尽量把对外暴露的属性设为 readonly
* 5.在确有必要时使用懒加载
* 6.尽量别用 KVC

API 设计
* 1.方法该设计成类方法还是实例方法？
* 2.分类中的属性及方法命名需要加前缀

协议
* 1.通过协议提供匿名对象

内存管理
* 1.在 dealloc 方法中只释放引用并解除监听
* 2.不要使用 retainCount

