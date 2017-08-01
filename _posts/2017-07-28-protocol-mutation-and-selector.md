---
layout: post
title:  "Swift | protocol mutation 与 selector"
date:   2017-07-28 15:47:20 +0800
categories: 技术
tags: iOS Swift
author: JonyFang
mathjax: true
---

* content
{:toc}


> 《Swifter Tips -- SELECTOR》笔记，原文链接：[SELECTOR](http://swifter.tips/selector/)
>
> 《Swifter Tips -- PROTOCOL MUTATING》笔记，原文链接：[将 PROTOCOL 的方法声明为 MUTATING](http://swifter.tips/protocol-mutation/)







## PROTOCOL MUTATING
----

Swift 的 mutating 关键字，是为了能在该方法中修改 struct 或 enum 的变量。

实例代码如下：

```swift
protocol Vehicle {
    var numberOfWheels: Int {get}
    var color: UIColor {get set}
    
    mutating func changeColor()
}

struct MyCar: Vehicle {
    let numberOfWheels = 4
    var color = UIColor.blue
    
    mutating func changeColor() {
        color = UIColor.red
    }
}
```

需要注意的是：使用` class `实现带有` mutating `的方法时，实现前不需要加` mutating`，class 可以随意更改自己的成员变量。


## Selector 
----

Objective-C 中 @selector 比较常用，例如：

- 设定 `target-action`
- 接收通知时的方法调用

`selector `是 Objective-C runtime 的概念，如果` selector `对应的方法只对 Swift 可见(使用关键字` private `或` fileprivate`)，会出现` unrecognized selector `的错误。

示例代码：

```swift
@objc private func callMe() {
    //...
}

NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector:#selector(callMe), userInfo: nil, repeats: true)
```

如果作用域中存在同名的两个方法，即使函数签名不同，也会编译不通过。正确的使用方法：

```swift
func commonFunc() {
    //
}

func commonFunc(input: Int) -> Int {
    return input
}

let method1 = #selector(commonFunc as ()->())
let method2 = #selector(commonFunc as Int->Int)

NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector:method1, userInfo: nil, repeats: true)
NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector:method2, userInfo: nil, repeats: true)
```

## 遗留问题
----

- @selector 与 NSSelectorFromString 的对比



----

欢迎转载，但请注明出处. [https://github.com/JonyFang/dev-notes](https://github.com/JonyFang/dev-notes)