---
layout: post
title: "UIKit力学模型入门"
date: 2013-06-15 15:11
comments: true
categories: 能工巧匠集
published: false
---

### 什么是UIKit动力学（UIKit Dynamics）

其实就是UIKit的一套动画和交互体系。我们现在进行UI动画基本都是使用CoreAnimation或者UIView animations。而UIKit动力学最大的特点是将现实世界动力驱动的动画引入了UIKit，比如重力，铰链连接，碰撞，悬挂等效果。一言蔽之，即是，将2D物理引擎引入了人UIKit。

目的当然是更加自然和炫目的UI动画效果，比如模拟现实的拖拽和弹性效果，放在以前如果单用iOS SDK的动画实现起来还是相当困难的，而在UIKit Dynamics的帮助下，复杂的动画效果可能也只需要很短的代码（基本100行以内...其实现在用UIView animation想实现一个不太复杂的动画所要的代码行数都不止这个数了吧）。总之，便利多多，配合UI交互设计，以前很多不敢想和不敢写（至少不敢自己写）的效果实现起来会非常方便，也相信在iOS7的时代各色使用UIKit动力学的应用的在动画效果肯定会上升一个档次。

### 那么，应该怎么做呢

#### UIKit动力学实现的结构

为了实现动力UI，需要注册一套UI行为的体系，之后UI便会按照预先的设定进行运动了。我们应该了解的新的基本概念有如下四个：

* UIDynamicItem：用来描述一个力学物体的状态，其实就是实现了UIDynamicItem委托的对象
* UIDynamicBehavior：动力行为的描述，用来指定UIDynamicItem应该如何运动，即定义适用的物理规则
* UIDynamicAnimator；动画的播放者，动力行为（UIDynamicBehavior）的容器，添加到容器内的行为将发挥作用
* ReferenceView：等同于力学参考系，如果你的初中物理不是语文老师教的话，我想你知道这是啥..只有当想要添加力学的UIView是ReferenceView的子view时，动力UI才发生作用。

光说不练假把式，来做点简单的demo吧。比如为一个view添加重力行为：

```objc
- (void)viewDidLoad
{
    [super viewDidLoad];
    
    UIView *aView = [[UIView alloc] initWithFrame:CGRectMake(100, 50, 100, 100)];
    aView.backgroundColor = [UIColor lightGrayColor];
    [self.view addSubview:aView];
    
    UIDynamicAnimator* animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];
    UIGravityBehavior* gravityBeahvior = [[UIGravityBehavior alloc] initWithItems:@[aView]];
    [animator addBehavior:gravityBeahvior];
    self.animator = animator;
}
```
运行结果，view开始受重力影响了：

![重力作用下的UIview](http://img.onevcat.com/2013/uikit-dynamics-gravity.gif)

代码很简单，以现在ViewController的view为参照系（ReferenceView），来初始化一个UIDynamicAnimator，然后分配并初始化一个动力行为，这里是UIGravityBehavior，将需要进行物理模拟的UIDynamicItem传入。`UIGravityBehavior`的`initWithItems:`接受的参数为包含id<UIDynamicItem>的数组，另外`UIGravityBehavior`实例还有一个`addItem:`方法接受单个的id<UIDynamicItem>。就是说，实现了UIDynamicItem委托的对象，都可以看作是被力学特性影响的，进而参与到计算中。UIDynamicItem委托需要我们实现`bounds`，`center`和`transform`三个属性，在UIKit Dynamics计算新的位置时，需要向Behavior内的item询问这些参数，以进行正确计算。iOS7中，UIView和UICollectionViewLayoutAttributes已经默认实现了这个接口，所以这里我们直接把需要重力的