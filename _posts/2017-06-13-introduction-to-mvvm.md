---
layout: post
title:  "objc.io 笔记 | MVVM 介绍"
date:   2017-06-13 13:54:18 +0800
categories: 技术
tags: objc.io 笔记
author: JonyFang
mathjax: true
---

* content
{:toc}

这一篇是对 Objc.io 中《MVVM 介绍》的总结，罗列了自己阅读过程中记录的点。也可以前往 Objc.io 查看原文。中文版:[《MVVM 介绍》](https://objccn.io/issue-13-1/)英文版:[ Introduction to MVVM ](http://www.objc.io/issue-13/mvvm.html)





## 典型 MVC 结构图

![MVC](https://github.com/JonyFang/dev-notes/blob/master/images/2017-06-13-mvc.jpg?raw=true)
![MVC](https://github.com/JonyFang/dev-notes/blob/master/images/2017-06-13-vvc-model.jpg?raw=true)

`Model `呈现数据，`View `呈现用户界面，而` View Controller `调节它两者之间的交互。

`MVC `的缺点：没有解决 iOS 应用中日益增长的`重量级视图控制器`的问题。`View Controller `中常包含一些`“表示逻辑”`，如将一个` NSDate `转换为一个格式化过的` NSString`.

----
## MVVM

**View Model 是什么**

![MVVM](https://github.com/JonyFang/dev-notes/blob/master/images/2017-06-13-mvvm.jpg?raw=true)

把`“表示逻辑”`相关的内容，放到` View Model `中。`View Model `位于` ViewController `与` Model `之间。

总结下 MVVM： 一个 MVC 的增强版，连接了视图和控制器，并将表示逻辑从` View Controller `中移出放到一个新的对象里，即` View Model `。MVVM 听起来很复杂，其实本质上即是一个优化的 MVC 架构。

那么，为什么用` MVVM `呢？

- 减少` ViewController `的复杂性
- 兼容当下的` MVC `架构
- 增加应用的可测试性（表示逻辑易于测试）

----
## MVVM 示例

列举一个 MVC 架构的例子，`Model `层名为` Person `，`View Controller `层名为` PersionViewController`。代码如下：

```objc
@interface Person : NSObject

- (instancetype)initwithFirstName:(NSString *)firstName lastName:(NSString *)lastName birthdate:(NSDate *)birthdate;

@property (nonatomic, readonly) NSString *firstName;
@property (nonatomic, readonly) NSString *lastName;
@property (nonatomic, readonly) NSDate *birthdate;

@end
```

在` PersonViewController `中应用` Person `Model.

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    [self setupViews];
}

- (void)setupViews {
    self.nameLabel.text = [NSString stringWithFormat:@"%@ %@", self.model.firstName, self.model.lastName];

    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"EEEE MMMM d, yyyy"];
    self.birthdateLabel.text = [dateFormatter stringFromDate:self.model.birthdate];
}
```

上面即是一个标准的` MVC `模式，下面使用一个` View Model `来增强它.

```objc
//PersonViewModel.h
@interface PersonViewModel : NSObject

- (instancetype)initWithPerson:(Person *)person;

@property (nonatomic, readonly) Person *person;

@property (nonatomic, readonly) NSString *nameText;
@property (nonatomic, readonly) NSString *birthdateText;

@end

//PersonViewModel.m
@implementation PersonViewModel

- (instancetype)initWithPerson:(Person *)person {
    self = [super init];
    if (!self) return nil;

    _person = person;
    if (person.salutation.length > 0) {
        _nameText = [NSString stringWithFormat:@"%@ %@ %@", self.person.salutation, self.person.firstName, self.person.lastName];
    } else {
        _nameText = [NSString stringWithFormat:@"%@ %@", self.person.firstName, self.person.lastName];
    }

    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"EEEE MMMM d, yyyy"];
    _birthdateText = [dateFormatter stringFromDate:person.birthdate];

    return self;
}

@end
```

在` View Controller `中应用改进后的` View Model `，只需如下这样就可以了.

```objc
- (void)viewDidLoad {
    [super viewDidLoad];

    self.nameLabel.text = self.viewModel.nameText;
    self.birthdateLabel.text = self.viewModel.birthdateText;
}
```

对比第一种方式，可以看到` View Controller `相比之前，变得更轻量了.

----
## 测试

在抽象` View Model `的过程中，我们尽可能得将`“表示逻辑”`移入` View Model `中。正因为` View Controller `中要处理的逻辑变少了，与之前相比较` View Controller `的测试变得容易了很多。同时` View Model `也易于测试。如下测试案例：

```objc
SpecBegin(Person)
    NSString *firstName = @"first";
    NSString *lastName = @"last";
    NSDate *birthdate = [NSDate dateWithTimeIntervalSince1970:0];
    
    it (@"should use firstName and lastName. ", ^{
        Person *person = [[Person alloc] initWithFirstName:firstName lastName:lastName birthdate:birthdate];
        PersonViewModel *viewModel = [[PersonViewModel alloc] initWithPerson:person];
        expect(viewModel.nameText).to.equal(@"first last");
    });

    it (@"should use the correct date format. ", ^{
        Person *person = [[Person alloc] initWithFirstName:firstName lastName:lastName birthdate:birthdate];
        PersonViewModel *viewModel = [[PersonViewModel alloc] initWithPerson:person];
        expect(viewModel.birthdateText).to.equal(@"Thursday January 1, 1970");
    });
SpecEnd
```

----
## 总结

上面的例子中，`Model `是不可变的，只要在初始化时指定` View Model `的属性即可。 而对于可变` Model `，需要使用一些绑定机制，以在` Model `改变时，`View Model `随之更新。同时，`View Model `发生变化时，对应的` View `也需要得到及时的更新。

简而言之，过程即为：**Model 变更 -> 更新对应 View Model -> 更新对应 View**。

在 iOS 中，我们可以通过 **KVO（Key Value Observation）**来处理过程中的绑定，但这样会需要大量的样板代码，可以使用[ ReactiveCocoa ](https://github.com/ReactiveCocoa/ReactiveCocoa)来作为替代。


**罗列下前面内容，讲到了下面这几个：**

- 如何从普通的MVC派生出MVVM
- MVVM相比较MVC的优势在哪
- MVVM中对于可变的 Model，如何进行实时绑定

----

欢迎转载，但请注明出处. [https://github.com/JonyFang/dev-notes](https://github.com/JonyFang/dev-notes)




