---
layout: post
title:  "Swift | 多元组、@autoclosure 与 ??"
date:   2017-07-31 11:00:20 +0800
categories: 技术
tags: iOS Swift
author: JonyFang
mathjax: true
---

* content
{:toc}


> 《Swift Tips - 多元组》笔记，原文链接：[多元组](http://swifter.tips/tuple/)
>
> 《Swift Tips - @AUTOCLOSURE 和 ??》笔记，原文链接：[@AUTOCLOSURE 和 ??](http://swifter.tips/autoclosure/)







## 多元组

交换元素的示例：

```swift
func swapMe<T>(inout a: T, inout b: T) {
    (a, b) = (b, a)
}
```

再一个示例，将` CGRect `在某一位置切分为两个区域：

Objective-C 中通过`传入指针`的方式，让获取需要的部分。原因是在 Objective-C 中返回值只能有一个所造成的。

```objc
//CGRectDivide(CGRect rect, CGRect *slice, CGRect *remainder, CGFloat amount, CGRectEdge edge)
CGRect rect = CGRectMake(0, 0, 100, 100);
CGRect small;
CGRect large;
CGRectDivide(rect, &small, &large, 20, CGRectMinXEdge);
```

如上得出的结果是，将` (0, 0, 100, 100) `的` rect `分割为` (0, 0, 20, 100) `的` small `及` (20, 0, 80, 100) `的` large `。

相应的，在 Swift 中可以通过多元组的方式来实现类似功能：

```swift
extension CGRect {
    func divide(atDistance: CGFloat, fromEdge: CGRectEdge) -> (slice: CGRect, remainder: CGRect)
}

let rect = CGRectMake(0, 0, 100, 100)
let (small, large) = rect.divide(20, fromEdge: .MinXEdge)
```


----
## @AUTOCLOSURE

` @autoclosure `的作用是把表达式自动封装为一个闭包。

使用` @autoclosure `前的示例代码：

```swift
func logIfTrue(predicate: () -> Bool) {
    if predicate() {
        print("True")
    }
}

//几种调用代码
logIfTrue({return 2 > 1})
logIfTrue({2 > 1})
logIfTrue{2 > 1}
```

使用` @autoclosure `后的示例代码：

```swift
func logIfTrue(@autoclosure predicate: () -> Bool) {
    if predicate() {
        print("True")
    }
}

//使用 @autoclosure 后的调用
logIfTrue(2 > 1)
```

前后的对比，写法更简便、语义表达更清楚些。这里，` @autoclosure `的作用是让 Swift 把` 2 > 1 `这个表达式自动转化为` () -> Bool `。


## 操作符 ??

`操作符 ??`作用：当左侧值非 nil 时，返回对应值；当左侧值为 nil 时，返回右侧值。示例代码如下：

```swift
var level: Int?
var startLevel = 1

var currentlevel = level ?? startLevel
```

`?? `的实现原理是什么呢？实际上，`??` 有两个版本：

```swift
func ??<T>(optional: T?, @autoclosure defaultValue: () -> T?) -> T?
func ??<T>(optional: T?, @autoclosure defaultValue: () -> T) -> T
```

前面的` startLevel `在使用时，实际被封装成了一个`() -> Int`，推测` ?? `的实现：

```swift
func ??<T>(optional: T?, @autoclosure defaultValue: () -> T) -> T {
    switch optional {
        case .Some(let value):
            return value
        case .None:
            return defaultValue()
        }
}
```

`??` 操作符的实现方法，避免了一些额外的开销。原因是，在 `optional` 非 `nil` 时，只需要直接返回 optional 解包后的值，没有用到默认值；该方法的好处是，将默认值的计算推迟到` optional `判定为` nil `之后。需要注意的是，`@autoclosure` 不支持带有参数的写法。


## 遗留问题 

- Objective-C 中返回值为什么只能有一个



