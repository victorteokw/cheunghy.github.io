---
layout: post
title: "Cartography解读"
date: 2014-10-12 19:10:44 +0800
comments: true
categories: swift iOS
---
随着长屏与大屏iPhone的出现，传统的界面布局方式已经显露出了弊端。因此在两年前iPhone5刚刚推出的时候，苹果把自动布局从OS X带到了iOS。但是，由于苹果一贯“大道至简”的作风，自动布局只有两个啰嗦的api。所以，对自动布局的封装非常的有必要。在使用上，由于今年大屏手机的出现，和酷似脚本语言的Swift语言诞生，自动布局已经变得必要且简单。Cartography是一个Swift语言编写的自动布局框架，帮助实现界面的开发者简化布局过程。如果不熟悉自动布局，不用担心，因为Cartography已经满足绝大多数需要，并不需要触及自动布局复杂的概念。关于详细的自动布局介绍，请看[这篇文章](http://answerhuang.duapp.com/index.php/2013/10/11/advanced-auto-layout-toolbox/)。
  
### 关于Cartography
[github](https://github.com/robb/Cartography/)有使用的介绍，这篇文章需要读者至少看过它的使用方式。
  
要读懂这篇文章，最好是看着它的代码听着我的解说会更容易一些。
### 入口函数layout
Swift支持重载。以参数中只有一个view的函数来举例。它的实现是这样：
```
public func layout(view: View, block: LayoutProxy -> ()) {
    block(LayoutProxy(view))

    view.car_updateAutoLayoutConstraints()
}
```
Cartography的实现者使用用户传来的view参数，生成了一个布局代理，然后传递到了用户写的闭包中。然后对用户传来的view调用了一个扩展方法。
  
这个car_updateAutoLayoutConstraints()仅仅是对强制布局方法的一个封装，因为iOS与OS X下是不一样的。在iOS中，仅仅是相当于调用view.layoutIfNeeded()。layoutIfNeeded()是UIView的实例方法，作用在于，至少在api的用户看起来，强制立即使用新的布局约束更新界面。
  
真正的向视图添加约束的语句是block(LayoutProxy(view))，这个block是由用户来实现，一般是这个样：
``` 
layout(view1, view2) { view1, view2 in
    view1.width   == (view.superview!.width - 50) * 0.5
    view2.width   == view1.width - 50
    view1.height  == 40
    view2.height  == view1.height
    view1.centerX == view.superview!.centerX
    view2.centerX == view1.centerX

    view1.top >= view.superview!.top + 20
    view2.top == view1.bottom + 20
}
```
这段代码中的view1和view2是LayoutProxy的实例。LayoutProxy的初始化方法是这样：
``` 
init(_ view: View) {
// ... ...
```
这个下划线的作用是，使用者不需要给参数加标签。举个例子就懂：LayoutProxy(view)，而不是LayoutProxy(view: view)。
  
这个类有很多常量，除了view是初始化时用户提供的，superview是一个optional，其他的都是enum枚举类型。Swift中枚举是很强大的。superview是计算变量，运行时检查父视图是否存在，如果存在，创建一个LayoutProxy的实例并返回，所以可以用同样的方式描述父试图。
  
### NSLayoutAttribute的对应
每一个枚举都要么遵守Property协议，要么遵守Compound协议。Property协议定义了一个关联的视图，和一个布局属性，NSLayoutAttribute的实例。Compound是复合的意思，它定义了一个Property数组。
  
什么属性是一个属性，什么属性是复合属性很清晰。Edge对应上下左右，横轴中心，纵轴中心，和基线这种位置性的。Dimension对应宽和高这种自身大小性的布局属性。复合属性Size封装了长和宽，Edges封装了上下左右，Point封装了中心点。
  
这些枚举的定义文件中，有大量的符号定义。其中有很强的规律性：
1. 等号和不等号一些是调用apply函数，一些是交换参数传递到执行的函数中。这么做可以减少重复，是良好的代码规范。关于apply函数放到后面去说，因为它是生成约束的最后一个步骤。
2. Property的返回一个NSLayoutConstraint，而Compound的返回一个NSLayoutConstraint的数组。
3. 加减乘除一定返回Expression。Expression是带有范型的结构体。coefficients像是两个数字的数组，其实是一个结构体。按照苹果的官方文档说法，一切约束都可以由y = ax + b来描述，x和y是两个`试图的属性`的关系，而a和b就是这两个因子。
  
可以这么说：Expression记录着属性之间的关系，而等号与不等号是向试图添加约束条件的方法。
  
### 为试图添加约束
整个项目中一共有两个apply方法。Compound的apply方法只是将复合属性分开来调用Property的apply方法。
  
#### 第一步：取消将AutoresizingMask转为自动布局
你知道AutoresizingMask是古老的布局技术，如果你听说过iOS6之前的弹簧铁柱。然而，代码创建的试图都有这个问题，所以有这步。
#### 第二步：找出公共父视图
最佳实践认为：约束应该添加到两个视图的公共父试图。助手函数commonSuperview做这一件事。commonSuperview函数优化得很好。
#### 第三步：判断这个约束是发生在1个还是2个视图中
如果是一个视图的长等于宽，那么没有第二个视图，也就没有第二个视图的属性，这时，将要代入苹果api的属性也就是NSLayoutAttribute.NotAnAttribute。
#### 第四步：创建约束
代码已经一目了然，所有的枚举，结构体，符号重载，都是为了这一瞬间。
#### 第五步：添加约束
创建了约束，大功告成，只差一步。现在添加了约束。
#### 第六步：返回约束
在复杂的布局中，用户会对约束有引用，所以用户可以捕捉。
#### 补充：优先级
为什么在这里补充呢？因为优先级符号在NSLayoutConstraint上运算，苹果给回Cartography这个实例，然后Cartography给回用户。这时，才能修改优先级。而苹果的api并不能传参优先级。而优先级也是NSLayoutConstraint少数可变的属性之一。
  
### 写在后面
在这篇文章写完的时候，Cartography是一个正在发展的优秀项目，以后可能有细微的变化。
由于我个人水平问题，这篇文章可能有些欠缺，欢迎一起讨论，我的微博在[这里](http://weibo.com/kaiyuz/)。
