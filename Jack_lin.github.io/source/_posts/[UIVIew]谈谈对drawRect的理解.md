---
title: 谈谈对drawRect的理解 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- iOS学习记录
tags: # 这里写的标签会自动汇集到 tags 页面上
- UIView
---

![疯狂](http://upload-images.jianshu.io/upload_images/1713024-83437bc2f696a82c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 写在前面
 `UIView`对于iOS开发来讲，再熟悉不过了。也正是因为这一点，我们可能会忽略`UIView`一些特有方法的理解和使用。今天，笔者主要整理一下对`drawRect `方法的理解和使用。<br>
 
 默认情况下，该方法在视图加载过程中不做任何人处理。当子类使用`Core Graphics`和`UIKit `绘制视图内容时就需要在该方法中添加绘制的代码。

### drawRect简介
`drawRect`方法在`UIView `的使用上起着十分关键的作用。不知道大家注意过没有，每一次创建`UIView`子类文件时候，会有自动带有已注释的`drawRect `方法，也许从这一点就能看出这个方法的重要性。

该方法定义在`UIView(UIViewRendering)`分类里面，望文生义，该方法完成视图的绘制。

### drawRect作用
`Only override drawRect: if you perform custom drawing.`

重绘作用：重写该方法以实现自定义的绘制内容

### drawRect调用场景
视图第一次显示的时候会调用。这个是由系统自动调用的，主要是在`UIViewController`中`loadView`和`viewDidLoad `方法调用之后；

如果在`UIView`初始化时没有设置rect大小，将直接导致`drawRect`不被自动调用；

该方法在调用`sizeThatFits`后被调用,所以可以先调用`sizeToFit`计算出`size`,然后系统自动调用`drawRect:`方法；

 通过设置`contentMode`属性值为`UIViewContentModeRedraw`,那么将在每次设置或更改`frame`的时候自动调用`drawRect:`;
 
直接调用`setNeedsDisplay`，或者`setNeedsDisplayInRect:`触发`drawRect:`，但是有个前提条件是`rect`不能为0;

### drawRect重绘方法定义
  `- (void)drawRect:(CGRect)rect; `:重写此方法，执行重绘任务;
  
 `- (void)setNeedsDisplay; `:标记为需要重绘，异步调用drawRect，但是绘制视图的动作需要等到下一个绘制周期执行，并非调用该方法立即执行;
 
 `- (void)setNeedsDisplayInRect:(CGRect)rect;`:标记为需要局部重绘，具体调用时机同上;

### drawRect使用注意事项
如果子类直接继承自`UIView`,则在`drawRect`  方法中不需要调用`super`方法。若子类继承自其他`View`类则需要调用`super`方法以实现重绘。

 若使用`UIView`绘图，只能在`drawRect:`方法中获取绘制视图的contextRef。在其他方法中获取的contextRef都是不生效的；
 
`drawRect:`方法不能手动调用，需要调用实例方法`setNeedsDisplay `或者`setNeedsDisplayInRect `,让系统自动调用该方法；

 若使用`CALayer`绘图，只能在`drawInContext :`绘制，或者在`delegate`方法中进行绘制，然后调用`setNeedDisplay`方法实现最终的绘制；
 
若要实时画图，不能使用gestureRecognizer，只能使用touchbegan等方法来掉用setNeedsDisplay实时刷新屏幕 ------这个阐述需要调整

`UIImageView`继承自`UIView`,但是`UIImageView`能不重写`drawRect`方法用于实现自定义绘图。具体原因如下图所示：

![Apple官方文档描述](http://upload-images.jianshu.io/upload_images/1713024-374336bff0e5c456.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 小结一下

上面的几个问题说的有些啰嗦了，总结一下需要掌握一下几点：

了解`drawRect`使用场景；<br>
哪些方法可以调用；<br>
了解何时进行重绘；<br>

### 参考文献
[drawRect参考](https://developer.apple.com/reference/uikit/uiview/1622529-drawrect?language=objc)

[setNeedsDisplay使用](https://developer.apple.com/reference/uikit/uiview/1622437-setneedsdisplay)

> 扫一扫下面的二维码，欢迎关注我的个人微信公众号攻城狮的动态（ID:iOSDevSkills），可在微信公众号进行留言，更多精彩技术文章，期待您的加入！一起讨论，一起成长！一起攻城狮！

![公众号](http://upload-images.jianshu.io/upload_images/1713024-14a3d919f462f399.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
