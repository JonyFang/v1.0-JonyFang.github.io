---
layout: post
title:  "笔记 | iOS 开发中的 Bitcode"
date:   2017-06-21 15:09:20 +0800
categories: 技术
tags: iOS
author: JonyFang
mathjax: true
---

* content
{:toc}

最近项目中使用到`阿里百川用户反馈`模块，直接引入` Cocoapods 库`，编译时无法编译通过。显示了下面这样的一条 Error 信息：

```objc
'~/Pods/UTDID/UTDID.framework/UTDID(AidManager.o)' does not contain bitcode. You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE), obtain an updated library from the vendor, or disable bitcode for this target. for architecture arm64
```






<br>
Error 信息大概的意思是：
<br>

> UTDID 中有文件未支持 Bitcode，要么让第三方库支持 Bitcode；要么关闭 Bitcode 功能.
<br>
如果只是想使用`阿里百川反馈模块`功能，直接关闭项目的` Bitcode `功能即可。

<br>
然而，到这里自然会产生一些疑问：

- `Bitcode `是什么？
- `Bitcode `的作用是什么？
<br>
借助这两个问题，对` Bitcode `做一个了解。

----
## Bitcode 是什么，有什么作用

![](https://github.com/JonyFang/dev-notes/blob/master/images/2017-06-21-enable-bitcode.jpeg?raw=true)
<br>
实际上` Xcode 7 `之后，在我们新建一个 iOS 项目时，`Bitcode `选项默认是设置为 YES 的。我们可以在` ”Build Settings” -> ”Enable Bitcode” `中看到这个设置项。
<br><br>
Apple 目前采用的`编译器工具链`是` LLVM`，我们在编辑器中通过` C/C++/Objective-c/Swift `等语言编写的程序，通过` LLVM 编译器`编译为各个芯片平台上的汇编指令，或可执行机器指令数据。而` Bitcode `则是位于编译前后之间的一种中间码。
<br><br>
`LLVM `的编译工作原理是先把项目程序源代码翻译为` Bitcode 中间码`；再根据不同目标机器芯片平台，转换为相应的汇编指令及机器码。这样的设计就让` LLVM `成为了一个编译器架构，可以轻而易举的在 LLVM 架构之上发明新的语言，以及在 LLVM 架构下支持新的 CPU 指令输出。虽然 Bitcode 仅仅只是一个`中间码`，不能在任何平台上运行，但它可以转化为任何被支持的` CPU 指令`，包括还没被发明的 CPU。
<br><br>
也就是说现在`打开 Bitcode 功能`，`提交一个 App `到 App Store。如果后面 Apple 推出了一款 CPU 全新设计的手机，在`苹果后台服务器`一样可以通过这个 App 的 Bitcode 编译转化为适用于新 CPU 的可执行程序，以供新手机用户下载运行这个 App。
<br><br>
[@Onevcat ](http://weibo.com/onevcat)在 [开发者所需要知道的 iOS 9 SDK 新特性](https://onevcat.com/2015/06/ios9-sdk/)，对 Bitcode 有这样的介绍。
<br>
> 给 App 瘦身的另一个手段是提交 Bitcode 给 Apple，而不是最终的二进制。Bitcode 是 LLVM 的中间码，在编译器更新时，Apple 可以用你之前提交的 Bitcode 进行优化，这样你就不必在编译器更新后再次提交你的 app，也能享受到编译器改进所带来的好处。Bitcode 支持在新项目中是默认开启的，没有特别理由的话，你也不需要将它特意关掉。

----
## Bitcode 支持哪些平台

列出` iOS`、`WatchOS`、`TVOS `及` macOS `对 Bitcode 的支持情况：

- 对于` iOS `平台，Bitcode 是`可选`的；
- 对于` WatchOS `及` TVOS`，Bitcode 是`必须开启`的；
- 对于` macOS`，是`不支持` Bitcode 的.

----
## Bitcode 的优缺点

**优点**

- `Bitcode `上传到` Apple 服务器`后，Apple 为安装 App 的目标设备进行`二进制优化`，减少安装包的`下载大小`;
- Apple 以后如果设计了采用新指令集的`新 CPU`，可以继续使用同一份` Bitcode `编译出`新 CPU `上执行的可执行文件，供新设备用户下载安装；


**缺点**

- 使用了` Bitcode `之后，用户安装的二进制不是开发者这边生成的，而是` Apple 服务器`经过优化生成的二进制，对于开发者来说，丢失了对应的调试符号信息


----
## 总结

由`阿里百川用户反馈模块`集成遇到的一个` Bitcode `问题。单纯对于`用户反馈`这项功能来说，很好奇的一点是，通过手动引入` Framework `没有遇到` Bitcode `不支持的问题；而通过` Cocoapods `引入，遇到了前面提到的` Bitcode `问题。考虑到项目的后期维护性，最终选用的解决方案是，通过` Cocoapods `引入库，同时关闭` Bitcode `的支持。


----

欢迎转载，但请注明出处. [https://github.com/JonyFang/dev-notes](https://github.com/JonyFang/dev-notes)



