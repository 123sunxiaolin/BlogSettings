---
title: iOS权限完整解决 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- iOS学习记录
tags: # 这里写的标签会自动汇集到 tags 页面上
- 系统权限
---
![完美](http://upload-images.jianshu.io/upload_images/1713024-55511373617d98aa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 前言
iOS开发中，权限问题不可避免；

 写了文章[iOS开发中的这些权限，你搞懂了吗？](http://www.jianshu.com/p/27e57922232b)和[[续]iOS开发中的这些权限，你搞懂了吗？](http://www.jianshu.com/p/f7172e24be4f)，介绍了系统涵盖的16种权限访问的原理和方法；
 
开源库[JLAuthorizationManager](https://github.com/123sunxiaolin/JLAuthorization)，整理并提供常用权限访问的便捷方法；

### 开源库基本使用
针对相册、蜂窝网络、相机、麦克风、通讯录、日历、提醒事项、定位、媒体资料库、语音识别、Siri等，可统一使用一下的方法入口：

```
/**
 请求权限统一入口

 @param authorizationType 权限类型
 @param authorizedHandler 授权后的回调
 @param unAuthorizedHandler 未授权的回调
 */
- (void)JL_requestAuthorizationWithAuthorizationType:(JLAuthorizationType)authorizationType
                                   authorizedHandler:(void(^)())authorizedHandler
                                 unAuthorizedHandler:(void(^)())unAuthorizedHandler;
```

如果你在开发过程中想使用健康数据的权限，请使用的下面的方法：

```

/**
 请求健康数据权限统一入口

 @param typesToShare 共享/写入共享数据类型集合
 @param typesToRead 读入共享数据类型集合
 @param authorizedHandler 授权后的回调
 @param unAuthorizedHandler 未授权的回调
 */
- (void)JL_requestHealthAuthorizationWithShareTypes:(NSSet*)typesToShare
                                          readTypes:(NSSet*)typesToRead
                                  authorizedHandler:(void(^)())authorizedHandler
                                unAuthorizedHandler:(void(^)())unAuthorizedHandler;

```

如果你想在项目中使用社交账号，请调用下面的方法：

```
/**
  请求社交账号访问权限

 @param authorizationType 权限类型
 @param options 请求账号时需要的配置信息(Facebook 和 腾讯微博不能为空)
 @param authorizedHandler 授权后的回调
 @param unAuthorizedHandler 未授权的回调
 @param errorHandler 产生错误的回调
 */
- (void)JL_requestAccountAuthorizationWithAuthorizationType:(JLAuthorizationType)authorizationType
                                                    options:(NSDictionary *)options
                                          authorizedHandler:(void(^)())authorizedHandler
                                        unAuthorizedHandler:(void(^)())unAuthorizedHandler
                                               errorHandler:(void(^)(NSError *error))errorHandler;
```

### 开源库使用的最低要求
 Xcode 8.0及以上；
 
 iOS 8.0及以上；

### 开源库的安装
 **Cocoapods**安装，在`Podfile`文件中添加：
```
 pod 'JLAuthorizationManager', '~> 1.0.0'
```

手动安装，将项目clone到本地，将`JLAuthorizationManager `文件夹拖至项目即可；

### 其他
更多详细使用可阅读`README`文件或者运行`Demo`程序；

支持`MIT`开源协议；

近期会添加开源库的功能，并且更新记录会在该文章记录。

### 如有问题
当你在使用过程中，存在问题，敬请文章中评论或者在微信公众号内给我留言；

如果你有好的改进方法，敬请`Pull Request`；

如果感觉还可以，那就敬请`Star`；

> 扫一扫下面的二维码，欢迎关注我的个人微信公众号攻城狮的动态（ID:iOSDevSkills），可在微信公众号进行留言，更多精彩技术文章，期待您的加入！一起讨论，一起成长！一起攻城狮！

