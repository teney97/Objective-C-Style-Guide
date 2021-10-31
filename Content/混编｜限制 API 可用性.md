## 混编｜限制 API 可用性

关键词：NS_SWIFT_UNAVAILABLE、NS_UNAVAILABLE、available

### Unavailable

* 使用 `NS_SWIFT_UNAVAILABLE` 宏使 Objective-C API 在 Swift 中不可用
* 使用 `NS_UNAVAILABLE` 宏使 Objective-C API 在 Objective-C 和 Swift 中都不可用

#### NS_SWIFT_UNAVAILABLE

在混编时，如果 Objective-C 中某些 API 不适用于 Swift，可以使用 `NS_SWIFT_UNAVAILABLE` 宏让这些 API 在 Swift 中不可用，并指定一个字符串错误信息提示调用者应该怎么做（例如用其它 API 替代）。当尝试在 Swift 中调用使用 `NS_SWIFT_UNAVAILABLE` 标记的 API 时会产生编译错误，并得到指定的错误提示。

```objectivec
+ (instancetype)collectionWithValues:(NSArray *)values
                             forKeys:(NSArray<NSCopying> *)keys
NS_SWIFT_UNAVAILABLE("Use a dictionary literal instead.");
```

#### NS_UNAVAILABLE

如果要让 Objective-C API 在  Objective-C 和 Swift 中都不可用，可以使用 `NS_UNAVAILABLE` 宏，但它不支持自定义的错误信息。

```objectivec
- (instancetype)init NS_UNAVAILABLE;
```

### Available

#### 标记可用性

```swift
// Objective-C
@interface MyViewController : UIViewController
- (void) newMethod API_AVAILABLE(ios(11), macosx(10.13));
@end

// Swift
@available(iOS 11, macOS 10.13, *)
func newMethod() {
    // Use iOS 11 APIs.
}
```

#### 检查可用性

```swift
// Objective-C
if (@available(iOS 11, *)) {
    // Use iOS 11 APIs.
} else {
    // Alternative code for earlier versions of iOS.
}

// Swift
if #available(iOS 11, *) {
    // Use iOS 11 APIs.
} else {
    // Alternative code for earlier versions of iOS.
}
```

### 参考

* [Apple｜Making Objective-C APIs Unavailable in Swift](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/making_objective-c_apis_unavailable_in_swift)
* [Apple｜Marking API Availability in Objective-C](https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization/marking_api_availability_in_objective-c)

