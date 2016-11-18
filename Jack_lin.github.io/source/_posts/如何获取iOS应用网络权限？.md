---
title: 如何获取iOS应用网络权限？ # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- iOS学习记录
tags: # 这里写的标签会自动汇集到 tags 页面上
- 系统权限
---

![疯狂](http://upload-images.jianshu.io/upload_images/1713024-83437bc2f696a82c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 问题描述

在iOS 10下 ，首次进入应用时，会有询问是否允许网络连接权限的的弹窗，为更好进行用户交互，需要在打开应用时获取应用禁用网络权限状态（状态分为：未知、限制网络、未限制网络），客户端根据不同的权限状态定制相应的人机交互。

#### 问题调研

针对请求应用网络权限可能存在的几种情形，操作与对应的状态都是笔者测试得到的，具体如下所示：

可能操作 | 关闭| 无线局域网|无线局域网&蜂窝|不进行操作|锁屏|解锁|按Home键
------------ | ------------- | ------------
权限状态 | Restricted  | NotRestricted | NotRestricted | Unknown|Unknown|恢复原始状态|保持原有状态

#### 解决问题
 使用`CoreTelephony.framework`框架下的`CTCellularData`类中的方法和属性进行解决,具体如下：


当联网权限的状态发生改变时，会在上述方法中捕捉到改变后的状态，可根据更新后的状态执行相应的操作。

```
CTCellularData *cellularData = [[CTCellularData alloc]init];
cellularData.cellularDataRestrictionDidUpdateNotifier =  ^(CTCellularDataRestrictedState state){
        //状态改变时进行相关操作
    };

```
 当查询应用联网权限时可以使用下面的方法：

```
CTCellularData *cellularData = [[CTCellularData alloc]init];
CTCellularDataRestrictedState state = cellularData.restrictedState;
    switch (state) {
        case kCTCellularDataRestricted:
            NSLog(@"Restricrted");
            break;
        case kCTCellularDataNotRestricted:
            NSLog(@"Not Restricted");
            break;
        case kCTCellularDataRestrictedStateUnknown:
            NSLog(@"Unknown");
            break;
        default:
            break;
}

```

#### 补充一下
 `CoreTelephony.framework`iOS7之前还是私有框架，框架内部提供还是私有API,但在iOS7之后该框架就成为公开的框架，大家可以尽情的使用了。

写这篇博客一方面是为了弥补前些日子写的博客[iOS开发中的这些权限，你搞懂了吗？](http://www.jianshu.com/p/27e57922232b)中的不足之处，另一方面是为了解决部分读者的疑惑，希望读者大人们多多支持！