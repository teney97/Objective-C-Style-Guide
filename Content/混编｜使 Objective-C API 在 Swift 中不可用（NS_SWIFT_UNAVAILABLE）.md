## 混编｜使 Objective-C API 在 Swift 中不可用

* 使用 NS_SWIFT_UNAVAILABLE 宏使 Objective-C API 在 Swift 中不可用
* 使用 NS_UNAVAILABLE 宏使 Objective-C API 在  Objective-C 和 Swift 中都不可用

在混编时，如果 Objective-C 中某些 API 不适用于 Swift，可以使用 NS_SWIFT_UNAVAILABLE 宏让这些 API 在 Swift 中不可用，并指定一个字符串错误信息提示调用者应该怎么做（例如用其它 API 替代），如下。这样尝试在 Swift 中调用这些 API 时会导致编译错误，并给予指定的错误提示。

```objectivec
+ (instancetype)collectionWithValues:(NSArray *)values
                             forKeys:(NSArray<NSCopying> *)keys
NS_SWIFT_UNAVAILABLE("Use a dictionary literal instead.");
```

如果要让 Objective-C API 在  Objective-C 和 Swift 中都不可用，可以使用 NS_UNAVAILABLE 宏，但它不支持自定义的错误信息。该宏可用于：

* 在 Objective-C 中实现单例时禁用 init、new 等 API
* 当子类想要缩小它继承自父类的 API 范围时，对不想继承的 API 使用 
* ......

```objectivec
- (instancetype)init NS_UNAVAILABLE;
+ (instancetype)new NS_UNAVAILABLE;
```

### 参考

* [Apple｜Making Objective-C APIs Unavailable in Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/making_objective-c_apis_unavailable_in_swift)

