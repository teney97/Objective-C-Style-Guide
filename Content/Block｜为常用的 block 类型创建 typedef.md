## 为常用的 block 类型创建 typedef

以 typedef 关键字重新定义块类型：

```objectivec
typedef int(^EOCSomeBlock)(BOOL flag, int value);
EOCSomeBlock block = ^(BOOL flag, int value) {
    // Implementation
}
```

好处有：

- 易读，定义、声明块方便 
  定义块变量语法与定义其他类型变量的语法不同，而定义方法参数所用的块类型语法又和定义块变量语法不同。这种语法很难记，所以以 typedef 关键字重新定义块类型，可令块变量用起来更加简单，起个更为易读的名字来表示块的用途，而把块的类型隐藏在其后面。

  ```objectivec
  // block 作为属性、变量的语法
  @property (nonatomic, copy) int(^block)(BOOL flag, int value);
  int(^block)(BOOL flag, int value);
  // block 作为方法参数、返回值类型的语法
  - (void)setBlock:(int (^)(BOOL flag, int value))block;
  - (int (^)(BOOL flag, int value))block;
  // 以 typedef 关键字重新定义块类型，可以统一语法，还可以隐藏 block 的具体类型
  @property (nonatomic, copy) EOCSomeBlock block;
  - (void)setBlock:(EOCSomeBlock)block;
  ```

- 重构块的类型签名时很方便 
  当修改 typedef 类型定义语句时，使用到这个类型定义的地方都无法编译，可逐个修复。而若是不用类型定义直接写块类型，修改的地方就更多，而且很容易忘掉其中一两处的修改而引发难于排查的 bug。

  > 这点你可以类比两个类遵循同一个协议，当协议中约定的方法名修改的时候，调用两个类的该协议方法的地方都无法编译，可逐个修复。而如果不用协议，方法都声明在各自类的头文件中，修改其中一个类是不会影响使用另一个类的地方的。

最好在使用块类型的类中定义这些 typedef，而且新类型名称还应该以该类名开头，以阐明块的用途。

> 与全局类型常量一样，如果 typedef block 放在头文件中，以类名开头是为了避免类型冲突，导致编译错误。比如：
>
> ```objectivec
> // In a header file
> typedef int(^HTBlock)(BOOL flag, int value);
> // In another header file
> typedef int(^HTBlock)(BOOL flag); // ❌ Typedef redefinition with different types ('int (^)(BOOL)' vs 'int (^)(BOOL, int)')
> ```

可以根据不同用途为同一个块签名定义多个类型别名。如果要重构的代码使用了块类型的某个别名，那么只需修改相应的 typedef 中的块签名即可，无须改动其他 typedef。

> 也就是说，即使两个块签名一样，但用途不同，你也不应该使用同一个类型别名来定义，不然重构的时候有你受的。

回到标题，**为常用的 block 类型创建 typedef，而不是所有**。

> 参考：《Effective Objective-C 2.0》第 38 条。

