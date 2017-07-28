---
layout: post
title:  "Swift | 柯里化"
date:   2017-07-28 11:00:20 +0800
categories: 技术
tags: iOS, Swift
author: JonyFang
mathjax: true
---

* content
{:toc}

> 《Swifter Tips -- 柯里化 (CURRYING)》笔记，原文链接：[柯里化 (CURRYING)](http://swifter.tips/currying/)

`柯里化(Currying)`，也就是把接收多个参数的方法进行变形，使其`更加灵活`的方法。






## 柯里化示例一，数字累计相加
----

```swift
func addTo(adder: Int) -> Int -> Int {
    return {
        num in
        return num + adder
    }
}

let addTwo = addTo(2) //addTwo: Int -> Int
let result = addTwo(6) //result = 8
```

从代码，我们能看到作用是：定义一个通用的函数，它接收与输入相加的数，并返回一个函数。返回的函数作为一个整体，进行下一次的计算。


## 柯里化示例二，比较大小
----

```swift
func greaterThan(comparer: Int) -> Int -> Bool {
    return { $0 > comparer }
}

let greaterThan10 = greaterThan(10);

greaterThan10(13)    // => true
greaterThan10(9)     // => false
```


## 柯里化示例三
----

```swift
protocol TargetAction {
    func performAction()
}

struct TargetActionWrapper<T: AnyObject>: TargetAction {
    weak var target: T?
    let action: (T) -> () -> ()

    func performAction() -> () {
        if let t = target {
            action(t)()
        }
    }
}

enum ControlEvent {
    case TouchUpInside
    case ValueChanged
}

class Control {
    var actions = [ControlEvent: TargetAction]()

    func setTarget<T: AnyObject>(target: T, action: (T) -> () -> (), controlEvent: ControlEvent) {
        action[controlEvent] = TargetActionWrapper(target: target, action: action)
    }

    func removeTargetForControlEvent(controlEvent: ControlEvent) {
        actions[controlEvent] = nil
    }

    func performActionForControlEvent(controlEvent: ControlEvent) {
        actions[controlEvent]?.performAction()
    }
}
```


## 柯里化的优点
----

- 可以用来量产相似方法
- 避免了很多重复代码
- 方便后面的维护



