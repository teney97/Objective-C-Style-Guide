### 多用类型常量，少用 #define 预处理指令

定义常量时建议使用 `类型常量`，不建议使用 `预处理指令`。

**Preferred:**

```objectivec
static const NSTimeInterval kAnimationDuration = 0.3;
```

**Not Preferred:**

```objectivec
#define ANIMATION_DURATION 0.3
```

**`预处理指令` 和 `类型常量` 的区别：**

|          | 预处理指令     | 类型常量                                                   |
| -------- | -------------- | ---------------------------------------------------------- |
|          | 简单的文本替换 |                                                            |
|          | 不包括类型信息 | 包括类型信息（可以清楚描述常量的含义，有助于编写开发文档） |
|          | 可被任意修改   | 不可被修改                                                 |
|          |                | 可以设置其使用范围                                         |
| 编译时刻 | 预编译         | 编译                                                       |
| 编译检查 | 没有           | 有                                                         |

**预处理指令**

```objectivec
#define ANIMATION_DURATION 0.3
```

1. 预处理指令会把源代码中的 ANIMATION_DURATION 都替换为 0.3。
2. 该常量没有类型信息，它应该为 NSTimeInterval 类型才合理。
3. 假设此指令声明在某个头文件中，那么所有引入了这个头文件的代码，其 ANIMATION_DURATION 在编译时都会被替换为 0.3。

**类型常量**

```objectivec
static const NSTimeInterval kAnimationDuration = 0.3;
```

1. 该常量 kAnimationDuration 包含类型信息，其为 NSTimeInterval 类型（包括类型信息）。
2. static 修饰符意味着该常量只在定义它的 .m 中可见（设置了其使用范围）。
3. const 修饰符意味着该常量不可修改（不可修改）。

**类型常量命名法** 

1. 如果常量局限于某 “编译单元”（也就是 .m 中），则命名前面加字母 `k`，比如 `kAnimationDuration`。
2. 如果常量在类之外可见，定义成了全局常量，则通常以 `类名` 作为前缀，比如 `EOCViewClassAnimationDuration`。

**局部类型常量**

```objectivec
static const NSTimeInterval kAnimationDuration = 0.3;
```
  1. 一定要同时使用 static 和 const 来声明。这样编译器就不会创建符号，而是像预处理指令一样，进行值替换。
2. 如果试图修改由 const 修饰的变量，编译器就会报错。
3. 如果不加 static，则编译器就会它创建一个 “外部符号 symbol”。此时如果另一个编译单元中也声明了同名变量，那么编译器就会抛出 “重复定义符号” 的错误：
```objectivec
duplicate symbol _kAnimationDuration in:
      EOCAnimatedView.o
      EOCOtherView.o
```
4. 局部类型常量不放在 “全局符号表” 中，所以无须用类名作为前缀。

**全局类型常量**

```objectivec
// In the header file
extern NSString *const EOCStringConstant;
	
// In the implementation file
NSString *const EOCStringConstant = @"VALUE";
```

1. 此类常量会被放在 “全局符号表” 中，这样才可以在定义该常量的编译单元之外使用。
2. const 位置不同则常量类型不同，以上为，定义一个指针常量 EOCStringConstant，指向 NSString 对象。也就是说，EOCStringConstant 不会再指向另一个 NSString 对象。
3. extern 是告诉编译器，在 “全局符号表” 中将会有一个名叫 EOCStringConstant 的符号。这样编译器就允许代码使用该常量。因为它知道，当链接成二进制文件后，肯定能找到这个常量。
4. 必须要定义，而且只能定义一次，通常定义在声明该常量的 .h 的对应的 .m 中。
5. 在实现文件生成目标文件时（编译器每收到一个 “编译单元” .m，就会输出一份 “目标文件” ），编译器会在 “数据段” 为字符串分配存储空间。链接器会把此目标文件与其他目标文件相链接，以生成最终的二进制文件。凡是用到 EOCStringConstant 这个全局符号的地方，链接器都能将其解析。
6. 因为符号要放在全局符号表里，所以常量命名需谨慎，为避免名称冲突，一般以类名作为前缀。

> 参考：《Effective Objective-C 2.0》第 4 条。
