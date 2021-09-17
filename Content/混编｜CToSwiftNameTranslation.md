### CToSwiftNameTranslation

### 前言

我们知道，在混编时，为了让 Swift 调用 Objective-C API 符合 Swift 的使用习惯，编译器会根据一些转换规则为 Objective-C 接口自动生成的 Swift 接口，并做了一些优化。如果你还不知道怎么查看生成的 Swift 接口，可以查看 [Tip：如何查看编译器为 Objective-C 接口生成的 Swift 接口？](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/%E6%B7%B7%E7%BC%96%EF%BD%9CTip%EF%BC%9A%E5%A6%82%E4%BD%95%E6%9F%A5%E7%9C%8B%E7%BC%96%E8%AF%91%E5%99%A8%E4%B8%BA%20Objective-C%20API%20%E7%94%9F%E6%88%90%E7%9A%84%20Swift%20API.md)。

Swift 和 Objective-C 的命名风格是有所不同，例如 Swift 的 API 是由基名和参数标签组成的，⽽ Objective-C 基本上只有参数标签，没有单独的基名，所以基名的信息会包含在第⼀个参数标签⾥，这也导致了 Objective—C 的方法名会显得略长一些。举个例子，在以下示例中，Objective-C API 的参数标签为 `previousMissionsFlownByAstronaut`，再看编译器将其转换成的 Swift API，`previousMissionsFlownByAstronaut` 被拆分成了 `基名 previousMissionsFlown `、`参数标签 by`、`参数名称 astronaut ` 三部分。当然这里将 `flownBy` 做为参数标签貌似更合适，只不过编译器没有这么智能啦，目前这个转换结果还是不错的，如果你想进一步优化 API，可以使用 `NS_SWIFT_NAME` 宏重命名 Objective-C API，可以参考 [为 Swift 重命名 Objective-C API（NS_SWIFT_NAME）](https://github.com/teney97/Objective-C-Style-Guide/blob/main/Content/混编｜为%20Swift%20重命名%20Objective-C%20API（NS_SWIFT_NAME）.md)，该文章中也举了这个例子。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1631880844639-1fc4165c-a629-498a-a27f-622faa39723d.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

看到这里，你是否好奇 “编译器会根据一些转换规则为 Objective-C 接口自动生成的 Swift 接口” 这些转换规则到底是哪些呢？

* 除了以上的  `by` 介词，还有哪些介词会被识别？
* 除了方法名，其它的如宏等等，是怎么转换的？

前方似乎有一片领域等着我们去探索。了解这些转换规则对我们有什么帮助呢？笔者最近就在思考一个问题，是不是不应该用 Swift 编程风格编写 Objective-C API，而是继续用 Objective-C 表达清晰但命名长的风格，以得到混编时在 Swift 中调用体验的提升。那么，为了良好的混编体验，我们是不是应该了解转换规则以编写出更规范的 Objective-C API 呢？再举个例子，笔者所在的团队之前刚开始混编的时候就遇到了各种问题，与该篇文章相关的问题就比如有 “在 Swift 中找不到 Objective-C API，因为名称变了”，这就是因为没有花时间就调研就直接开始混编的后果，导致开发中碰到各种阻碍。

那么，我们该从哪里得知这些转换规则，总不能自己去写 demo 探索把，Apple 是肯定会给清单的，在周报小伙伴的帮助下，笔者找到了：

* [CToSwiftNameTranslation](https://github.com/apple/swift/blob/main/docs/CToSwiftNameTranslation.md)
* [CToSwiftNameTranslation-OmitNeedlessWords](https://github.com/apple/swift/blob/main/docs/CToSwiftNameTranslation-OmitNeedlessWords.md)

那么，开始探索把！待续。。。



### 参考

* [Apple - CToSwiftNameTranslation](https://github.com/apple/swift/blob/main/docs/CToSwiftNameTranslation.md)
* [Apple - CToSwiftNameTranslation-OmitNeedlessWords](https://github.com/apple/swift/blob/main/docs/CToSwiftNameTranslation-OmitNeedlessWords.md)
* 
