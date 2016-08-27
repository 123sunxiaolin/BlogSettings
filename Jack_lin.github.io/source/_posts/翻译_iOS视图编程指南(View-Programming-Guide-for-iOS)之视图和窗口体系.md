
---
title: 翻译_iOS视图编程指南之视图和窗口体系 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- 翻译
tags: # 这里写的标签会自动汇集到 tags 页面上
- iOS
- English
---


![图片源自网络](http://upload-images.jianshu.io/upload_images/1713024-5d01f6c3c909513a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[官方最新：View Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009503-CH1-SW2)

## 前言

前些日子，我发布一个苹果官方文档的[翻译](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009503-CH1-SW2)，之后就有不少同学朋友问我：`翻译苹果官方文档能做什么，开发过程用到的时候很少,浪费时间，还又没什么用。`今天，刚好有时间，就在此申明一下翻译苹果官方文档的实质作用：

 首先，翻译官方文档可以提高自身英语阅读能力和理解能力，增大自己的词汇量，良好的英语基础会让工作效率更上一层楼的；
 
 其次，对于iOS开发而言，官方文档可以让你更好地理解每一个技术点实现的基本原理，知其然更要知其所以然，这样对iOS开发的进阶者和初学者都有很大的帮助；
 
最后，翻译官方文档可以让你更加全神贯注，写代码的过中会出现分神、思维混沌等现象，这时候翻译点英文资料可以让你静下心来，我已试过，很有帮助；

还有一点，苹果官方文档实属繁多，全部按部就班的翻译会耗时很长。为提高此翻译文章实质作用，特恳请大家能给一些指引，大家可以把日常工作用到的章节私信评论给我，我用业余时间把这些急需的章节翻译出来，贡献给大家，当然，翻译的文章难免有不足的地方，还望各位好友指出，以提高文章质量。


## 视图和窗口体系结构

视图和窗口呈现应用的交互界面并且处理交互事件。UIKit和其他系统框架提供大量可以使用而很少改动或无需改动的视图。你也可以在与标准视图呈现内容不同的地方设置自定义视图。

无论你是使用系统视图还是自定义视图，都需要理解由[UIVIew](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/cl/UIView)和[UIWindow](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIWindow_Class/index.html#//apple_ref/occ/cl/UIWindow)类所提供的基础结构。这些类提供复杂的设施来管理视图的布局和显示。理解这些设施是如何工作的对于确保在应用发生变化时视图可以正常工作是非常重要的。

#### 视图结构的基本原理

 表面上，你可能想去做的就是处理视图对象(UIView的子类).一个视图对象规定了视图上矩形区域，并且在矩形区域上处理绘画和触摸事件。
 
 视图也可以是其他一些视图的父类，协调那些视图的位置和尺寸。UIView的大部分工作用于管理视图之间的关系，但也可根据自己的需要自定义视图默认的行为。
 
视图与核心动画层合力处理视图内容修改和动画显示。在UIKit的每个视图都是由一个图层对象(通常都是 [CALayer](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CALayer_class/index.html#//apple_ref/occ/cl/CALayer)的子类)支持，这些图层管理视图的存储回存以及处理视图相关的动画。大部分的操作都得通过UIView的接口。然而，在那些你需要的控制远多于视图渲染和动画行为的情形下，你需要通过图层来执行相应的操作。

为理解视图和图层的关系，下面的例子会对你有所帮助。图1-1展示了从[视图切换](https://developer.apple.com/library/ios/samplecode/ViewTransitions/Introduction/Intro.html#//apple_ref/doc/uid/DTS40007411)例子应用到底层核心动画层的关系。

应用中的视图包括窗口(本身也是视图)，一个作为视图容器的UIView对象，一个图片视图，一个展示控制的工具条，一个条按钮项(它本身不是视图，但他管理内部的视图)。*每个视图都有一个响应图层，并且可以通过视图的 [layer](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/instp/UIView/layer)属性访问到*其中，由于条按钮项不是视图，故不能直接访问它的 [layer](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/instp/UIView/layer)属性。在这些图层对象的后面是核心动画渲染对象和用于管理屏幕具体像素的硬件缓冲区。

![图1-1例子应用视图的体系结构](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Art/view-layer-store.jpg)

使用核心动画图层对象对于性能提升有重要的意义。尽可能少的调用视图对象的绘制代码，一旦代码被调用，就会被核心动画缓存下来，以便以后尽可能的复用。复用已渲染好的内容可以消除更新视图所带来的高消耗的绘制周期。在动画过程中，复用已存在的内容是相当重要的。这种复用机制与创建新的内容相比，消耗的成本更低。

#### 视图层次和子视图的管理

 一个视图在呈现自身内容之外，还可以作为其他视图的容器。当一个视图包含另一个视图时，两个视图间的父子关系就创建出来了。在关系中，孩子视图就是子视图，父亲视图就是超视图。这种关系的创建对于应用的虚拟外表和行为具有重要的意义。
 
表面上，子视图掩盖全部或部分父视图的内容。如果子视图是完全不透明的，有子视图组成的区域将会完全掩盖父视图相应地区域。如果子视图部分透明，在屏幕显示之前，父视图和子视图的内容就会混合在一起。每一个父视图都将子视图存储在一个有序的数组中，这个顺序影响着每个子视图可视度。如果两个兄弟视图相互重叠，最后加入的视图将会最先显示。

父子视图的关系也影响着一些视图行为。改变父视图大小会产生波浪作用，导致子视图的位置和尺寸也随之变化。当父视图的尺寸发生变化时，使用视图的调整功能以恰当的配置视图。另一些影响子视图的变化有：隐藏父视图、改变父视图的透明度、将数学变化应用到父视图的坐标系统中。

在视图层次中管理视图决定着你的应用是如何响应事件的。当在特定视图中发生触摸事件时，系统将会把带有触摸信息的事件对象直接发送到视图的处理机制中。

然而，如果视图没有处理特定的触摸事件时，它将会把事件对象传送到父视图。如果父视图没有处理事件，将会把事件对象传递到父视图的父视图，以此类推，直到响应链。特定的视图也会将事件对象传递到介于中间的响应对象，例如视图控制器。如果没有对象处理该事件，最终达到抛弃它的应用对象。

#### 视图绘制周期

视图类使用一种按需绘画模式呈现内容。当视图第一次出现在屏幕上，系统将会请求绘制其内容。系统捕获内容的快照，并将此快照作为视图的虚拟显示。

如果你从不想改变视图内容，那么视图的绘制代码可能从不会再次调用。快照被复用在包括视图在内的大部分操作。如果你改变了这个内容，你通知系统视图已发生改变。视图将会重复绘制视图和捕获快照的过程。

 当你视图的内容改变时，你没有直接重新绘制这些改变。相反，你可以使用[setNeedsDisplay](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/instm/UIView/setNeedsDisplay)或者[setNeedsDisplayInRect:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/instm/UIView/setNeedsDisplayInRect:)方法使你的视图失效。这些方法会告诉系统这些已改变内容的视图需要在下次机会重新绘制。系统直到当前运行循环结束才进行任何绘制操作。

#### 写在最后

这篇文章翻译很长时间，中间总是断断续续的，今天终于完成了，心里石头也算是放下了。

通过翻译文章，一方面让自己重新学习了一下以前的知识，一方面，也锻炼了自己的英语翻译的能力。虽然翻译水平很low吧，但我还是会坚持下去的，加油！

