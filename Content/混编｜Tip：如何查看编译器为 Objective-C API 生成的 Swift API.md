## 混编｜Tip：如何查看编译器为 Objective-C 接口生成的 Swift 接口？

在混编时，为了让 Swift 调用 Objective-C API 符合 Swift 的使用习惯，我们就要关注编译器为 Objective-C 接口自动生成的 Swift 接口，然后对质量不高的 API 进行优化。

那么，如何查看编译器为 Objective-C 接口生成的 Swift 接口呢？

选择你要查看的 Objective-C 头文件，点击左上角的 Related Items 按钮，将光标移动到 Generated Interface 那一栏，右边就会出现不同版本的 Swift 接口文件。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1629993920888-571f8e8d-8ee1-44f6-82f2-3e1a510c1c4d.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

选择一个点开后，就可以查看对应版本的 Swift 接口文件了。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1629993923672-6e2b7ccc-5c1a-4f88-9bf1-3ebce00c1d34.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

掌握这个技巧很重要，它能帮助我们理解编译器为 Objective-C 接口生成的 Swift 接口是什么样的，以及帮助我们更好地优化 Objective-C API。

### 参考

* [Apple｜WWDC20 10680 - Refine Objective-C frameworks for Swift](https://developer.apple.com/videos/play/wwdc2020/10680/)
