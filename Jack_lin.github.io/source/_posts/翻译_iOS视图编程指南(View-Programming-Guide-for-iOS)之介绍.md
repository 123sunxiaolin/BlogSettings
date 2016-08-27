---
title: 翻译_iOS视图编程指南之介绍 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- 翻译
tags: # 这里写的标签会自动汇集到 tags 页面上
- iOS
- English
---

[官方最新：View Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009503-CH1-SW2)
## 介绍

#### 关于窗口和视图

在iOS中，你可以使用窗口和视图将你应用的内容呈现在屏幕上。窗口本身是不具备呈现可视化内容的功能的，但它可以用作装有应用视图的容器。视图可以规定在窗口的某一部分显示特定的内容。例如，你可能需要显示图片、文本、图形或者一些组合的视图。同时，你也可以使用视图去组织和管理其他的视图。

#### 概览

每一个应用都至少有一个窗口和视图用以呈现内容，UIKit和其他的系统框架会提供一些预定义的视图用来呈现内容，这些视图从简单的按钮、文本标签到更加复杂的列表视图、选择器视图和滚动视图。如果这些还是不能满足你的需要，你可以自定义视图以及自我管理绘画和事件处理。

#### 视图管理应用可视化的内容

每一个视图都是UIView类的实例或者子类，视图在应用的窗口中负责管理矩形的区域。视图主要负责绘制内容、处理多点触摸事件、管理姿势图的布局.其中,绘制内容包括使用 Core Graphics、 OpenGL ES，以及UIKit的技术在特定矩形区域内绘制几何图形、图片以及文本。

视图可以在矩形区域内响应触摸事件、手势识别，甚至可以直接处理触摸事件。在视图层次中，父视图负责动态定位和规范子视图，这种动态改变子视图的能力可以使视图更好适应不断变化的状态，比如交互旋转和动画。

你可以将试图视为搭积木。用这些组合来构建属于你的人机交互，而不是只用一个视图显示所有的内容，你通常需要几个视图来构建视图层次。视图层次中的每个视图都是你所构建用户交互中特定的一部分，并通常为特殊类型内容所优化的（各司其职）。例如，UIKit就有用以显示文本、图片和其他类型内容的特定视图。

`相关章节：`[视图和窗口结构](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW1)、[视图](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/CreatingViews/CreatingViews.html#//apple_ref/doc/uid/TP40009503-CH5-SW1)

#### 窗口可协调视图的显示

窗口是[UIWindow](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIWindow_Class/index.html#//apple_ref/occ/cl/UIWindow)的实例用以呈现整个应用的用户交互。

窗口用视图（视图控制器）管理与可视化视图层次的交互和改变。大多数，应用的窗口从不发生改变，窗口一旦创建便保持不变，只有在窗口上的视图发生变化。

每个应用至少有一个窗口用以呈现设备主屏幕上的用户交互。如果有外置屏幕接入设备，应用会创建第二个窗口显示相应的内容。
`相关章节：`[窗口](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/CreatingWindows/CreatingWindows.html#//apple_ref/doc/uid/TP40009503-CH4-SW1)
####  动画可提供用户人机交互的反馈

动画可以将视图层次的改变可视化反馈给用户。系统规定了用以不同组织视图中呈现模态视图和过渡的标准动画。

然而，动画的许多属性也可以直接用来动画。例如，通过动画，你可以改变视图的透明度、屏幕上位置、尺寸、背景或者其他属性。如果你想直接操作视图下层的核心动画层对象，同样可以呈现出其他的动画形式。

`相关章节：`[动画](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/AnimatingViews/AnimatingViews.html#//apple_ref/doc/uid/TP40009503-CH6-SW1)
#### Interface Builder的作用

Interface Builder是一款用来图形化构建和配置应用的窗口和视图。

使用Interface Builder，你会将你的视图存放在nib文件中，这种文件是一种存储视图和其他对象原始版本关系的资源文件，一旦在runtime中加载nib文件，nib文件中的对象就会重新组成可代码操作的具体对象。

Interface Builder极大的简化了创建应用交互界面的工作。因为在iOS机制中支持Interface Builder和nib文件混合使用的，并且很容易就可以将nib文件融合到应用程序的设计中。

`版权所有 ©2014 苹果 `
