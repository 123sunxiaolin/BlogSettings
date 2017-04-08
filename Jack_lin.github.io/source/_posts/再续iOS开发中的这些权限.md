---
title: 再续iOS开发中的这些权限 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- iOS学习记录
tags: # 这里写的标签会自动汇集到 tags 页面上
- 系统权限
---

![疯狂](http://upload-images.jianshu.io/upload_images/1713024-83437bc2f696a82c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 前言 

上篇文章[iOS开发中的这些权限，你搞懂了吗？](http://www.jianshu.com/p/27e57922232b)介绍了一些常用权限的获取和请求方法，知道这些方法的使用基本上可以搞定大部分应用的权限访问的需求。但是，这些方法并不全面，不能涵盖住所有权限访问的方法。

 So，笔者在介绍一下剩下的几种权限的访问方法和一些使用上的注意事项，希望能给大家的开发过程带来一丝便利。

最后，笔者将经常使用的权限请求方法封装开源库[JLAuthorizationManager](https://github.com/123sunxiaolin/JLAuthorizationManager)送给大家，欢迎大家**pull request** 和 **star**~~

### 权限
语音识别；

媒体资料库/Apple Music;
 
Siri;

健康数据共享；

蓝牙；

住宅权限（HomeKit）;

社交账号体系权限；

活动与体能训练记录;

广告标识；

### 语音识别
引入头文件： **@import Speech;**

首先判断当前应用所处的权限状态，若当前状态为**NotDetermined**（未确定），此时，需要调用系统提供的请求权限方法，同时也是触发系统弹窗的所在点；

该权限涉及到的类为** SFSpeechRecognizer**，具体代码如下：

```
- (void)p_requestSpeechRecognizerAccessWithAuthorizedHandler:(void(^)())authorizedHandler
                                         unAuthorizedHandler:(void(^)())unAuthorizedHandler{
    
    SFSpeechRecognizerAuthorizationStatus authStatus = [SFSpeechRecognizer authorizationStatus];
    if (authStatus == SFSpeechRecognizerAuthorizationStatusNotDetermined) {
         //调用系统提供的权限访问的方法
        [SFSpeechRecognizer requestAuthorization:^(SFSpeechRecognizerAuthorizationStatus status) {
            if (status == SFSpeechRecognizerAuthorizationStatusAuthorized) {
                dispatch_async(dispatch_get_main_queue(), ^{
                     //授权成功后
                    authorizedHandler ? authorizedHandler() : nil;
                });
            }else{
                dispatch_async(dispatch_get_main_queue(), ^{
                    //授权失败后
                    unAuthorizedHandler ? unAuthorizedHandler() : nil;
                });
            }
        }];
        
    }else if (authStatus == SFSpeechRecognizerAuthorizationStatusAuthorized){
        authorizedHandler ? authorizedHandler() : nil;
    }else{
        unAuthorizedHandler ? unAuthorizedHandler() : nil;
    }
}
```

需要注意的是，调用`requestAuthorization`方法的block回调是在任意的**子线程**中进行的，如果你需要在授权成功后刷新UI的话，需要将对应的方法置于**主线程**中进行，笔者将上述方法默认在主线程中进行。后续权限请求方法与此类似，不再赘述。

在**info.plist**添加指定的配置信息，如下所示：
![Speech Recognizer](http://upload-images.jianshu.io/upload_images/1713024-9ea85da828de805b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 媒体资料库/Apple Music
 导入头文件**@import MediaPlayer;**
 
使用类**MPMediaLibrary**进行权限访问，代码如下；


```
- (void)p_requestAppleMusicAccessWithAuthorizedHandler:(void(^)())authorizedHandler
                                   unAuthorizedHandler:(void(^)())unAuthorizedHandler{
    MPMediaLibraryAuthorizationStatus authStatus = [MPMediaLibrary authorizationStatus];
    if (authStatus == MPMediaLibraryAuthorizationStatusNotDetermined) {
        [MPMediaLibrary requestAuthorization:^(MPMediaLibraryAuthorizationStatus status) {
            if (status == MPMediaLibraryAuthorizationStatusAuthorized) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    authorizedHandler ? authorizedHandler() : nil;
                });
            }else{
                dispatch_async(dispatch_get_main_queue(), ^{
                    unAuthorizedHandler ? unAuthorizedHandler() : nil;
                });
            }
        }];
    }else if (authStatus == MPMediaLibraryAuthorizationStatusAuthorized){
         authorizedHandler ? authorizedHandler() : nil;
    }else{
        unAuthorizedHandler ? unAuthorizedHandler() : nil;
    }
}
```

在**info.plist**添加指定的配置信息，如下所示：
![Media](http://upload-images.jianshu.io/upload_images/1713024-576f14dca41175cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Siri
导入头文件**@import Intents;**；

与其他权限不同的时，使用**Siri**需要在Xcode中**Capabilities**打开**Siri**开关，Xcode会自动生成一个**xx.entitlements**文件，若没有打开该开关，项目运行时会报错。

实现代码如下：

```
- (void)p_requestSiriAccessWithAuthorizedHandler:(void(^)())authorizedHandler
                             unAuthorizedHandler:(void(^)())unAuthorizedHandler{
    INSiriAuthorizationStatus authStatus = [INPreferences siriAuthorizationStatus];
    if (authStatus == INSiriAuthorizationStatusNotDetermined) {
        [INPreferences requestSiriAuthorization:^(INSiriAuthorizationStatus status) {
            if (status == INSiriAuthorizationStatusAuthorized) {
                dispatch_async(dispatch_get_main_queue(), ^{
                     authorizedHandler ? authorizedHandler() : nil;
                });
            }else{
                dispatch_async(dispatch_get_main_queue(), ^{
                    unAuthorizedHandler ? unAuthorizedHandler() : nil;
                });
            }
        }];
        
    }else if (authStatus == INSiriAuthorizationStatusAuthorized){
        authorizedHandler ? authorizedHandler() : nil;
    }else{
        unAuthorizedHandler ? unAuthorizedHandler() : nil;
    }
}
```

### 健康数据共享
导入头文件**@import HealthKit;**


 健康数据共享权限相对其他权限相对复杂一些，分为写入和读出权限.
 

 在Xcode 8中的`info.plist`需要设置以下两种权限：

```
1、Privacy - Health Update Usage Description
2、Privacy - Health Share Usage Description
```

 具体实现代码：

```
//设置写入/共享的健康数据类型
- (NSSet *)typesToWrite {
    HKQuantityType *stepType = [HKObjectType quantityTypeForIdentifier:HKQuantityTypeIdentifierStepCount];
    HKQuantityType *distanceType = [HKObjectType quantityTypeForIdentifier:HKQuantityTypeIdentifierDistanceWalkingRunning];
    return [NSSet setWithObjects:stepType,distanceType, nil];
}

//设置读写以下为设置的权限类型：
- (NSSet *)typesToRead {
   HKQuantityType *stepType = [HKObjectType quantityTypeForIdentifier:HKQuantityTypeIdentifierStepCount];
    HKQuantityType *distanceType = [HKObjectType quantityTypeForIdentifier:HKQuantityTypeIdentifierDistanceWalkingRunning];
    return [NSSet setWithObjects:stepType,distanceType, nil];
}

//需要确定设备支持HealthKit
if ([HKHealthStore isHealthDataAvailable]) {
        return;
    }
HKHealthStore *healthStore = [[HKHealthStore alloc] init];
    NSSet * typesToShare = [self typesToWrite];
    NSSet * typesToRead = [self typesToRead];
    [healthStore requestAuthorizationToShareTypes:typesToShare readTypes:typesToRead completion:^(BOOL success, NSError * _Nullable error) {
        if (success) {
            dispatch_async(dispatch_get_main_queue(), ^{
                NSLog(@"Health has authorized!");
            });
        }else{
            dispatch_async(dispatch_get_main_queue(), ^{
                NSLog(@"Health has not authorized!");
            });
        }
    }];
```

### 蓝牙
需要导入头文件**@import CoreBluetooth;**

蓝牙的权限检测相对其他会复杂一些，需要在代理中检测蓝牙状态；

 获取蓝牙权限：
 

```
- (void)checkBluetoothAccess {
    CBCentralManager *cbManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil];
    CBManagerState state = [cbManager state];
    if(state == CBManagerStateUnknown) {
        NSLog(@"Unknown!");
    }
    else if(state == CBManagerStateUnauthorized) {
         NSLog(@"Unauthorized!");
    }
    else {
        NSLog(@"Granted!");
    }
}

- (void)centralManagerDidUpdateState:(CBCentralManager *)central {
//这个代理方法会在蓝牙权限状态发生变化时被调用，并且可以根据不同的状态进行相应的修改UI或者数据访问的操作。
}

```

 请求蓝牙权限

```
- (void)requestBluetoothAccess {
    CBCentralManager *cbManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil];
//该方法会显示用户同意的弹窗
    [cbManager scanForPeripheralsWithServices:nil options:nil];
}

```

### 住宅权限（HomeKit）
需导入头文件**@import HomeKit;**

`HomeKit`请求权限的方法如下：


```
- (void)requestHomeAccess {
    self.homeManager = [[HMHomeManager alloc] init];
//当设置该代理方法后，会请求用户权限
    self.homeManager.delegate = self;
}

- (void)homeManagerDidUpdateHomes:(HMHomeManager *)manager {
    if (manager.homes.count > 0) {
        // home的数量不为空，即表示用户权限已通过
    }
    else {
        __weak HMHomeManager *weakHomeManager = manager; // Prevent memory leak
        [manager addHomeWithName:@"Test Home" completionHandler:^(HMHome *home, NSError *error) {
            
            if (!error) {
               //权限允许
            }
            else {
                if (error.code == HMErrorCodeHomeAccessNotAuthorized) {
                   //权限不允许
                }
                else {
                    //处理请求产生的错误
                }
            }
            
            if (home) {
                [weakHomeManager removeHome:home completionHandler:^(NSError * _Nullable error) {
                    //移除Home
                }];
            }
        }];
    }
}

```

### 社交账号体系权限
导入头文件**@import Accounts;**

获取对应的权限：

```
- (void)checkSocialAccountAuthorizationStatus:(NSString *)accountTypeIndentifier {

    ACAccountStore *accountStore = [[ACAccountStore alloc] init];
    ACAccountType *socialAccount = [accountStore accountTypeWithAccountTypeIdentifier:accountTypeIndentifier];
    if ([socialAccount accessGranted]) {
        NSLog(@"权限通过了");
    }else{
         NSLog(@"权限未通过！");
 }
}
```

 `accountTypeIndentifier ` 可以是以下类型：

```
ACCOUNTS_EXTERN NSString * const ACAccountTypeIdentifierTwitter NS_AVAILABLE(NA, 5_0);
ACCOUNTS_EXTERN NSString * const ACAccountTypeIdentifierFacebook NS_AVAILABLE(NA, 6_0);
ACCOUNTS_EXTERN NSString * const ACAccountTypeIdentifierSinaWeibo NS_AVAILABLE(NA, 6_0);
ACCOUNTS_EXTERN NSString * const ACAccountTypeIdentifierTencentWeibo NS_AVAILABLE(NA, 7_0);
ACCOUNTS_EXTERN NSString * const ACAccountTypeIdentifierLinkedIn NS_AVAILABLE(NA, NA);
```

请求对应的权限：

```
- (void)requestTwitterAccess {
    ACAccountStore *accountStore = [[ACAccountStore alloc] init];
    ACAccountType *accountType = [accountStore accountTypeWithAccountTypeIdentifier:accountTypeIdentifier];
   
    [accountStore requestAccessToAccountsWithType: accountType options:nil completion:^(BOOL granted, NSError *error) {
        dispatch_async(dispatch_get_main_queue(), ^{
           if(granted){
                 NSLog(@"授权通过了");
            }else{
                 NSLog(@"授权未通过");
            }
        });
    }];
}

```

### 活动与体能训练记录
导入头文件**@import CoreMotion;**

具体实现代码：

```
//访问活动与体能训练记录
    CMMotionActivityManager *cmManager = [[CMMotionActivityManager alloc] init];
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    [cmManager startActivityUpdatesToQueue:queue withHandler:^(CMMotionActivity *activity) {
        
        //授权成功后，会进入Block方法内，授权失败不会进入Block方法内
    }];
```

### 广告标识

导入头文件**@import AdSupport;**

获取广告标识的权限状态：

```
 BOOL isAuthorizedForAd = [[ASIdentifierManager sharedManager] isAdvertisingTrackingEnabled];
```
在使用`advertisingIdentifier`属性前，必须调用上述方法判断是否支持，如果上述方法返回值为`NO`，则`advertising ID`访问将会受限。

### 小结一下
通过以上两篇文章的整理，有关iOS系统权限问题的处理基本上涵盖完全了；

 并不是所有的权限访问都有显式的调用方法，有些是在使用过程中进行访问的，比如`定位权限`、`蓝牙共享权限`、`Homekit权限`、`活动与体能训练权限`，这些权限在使用时注意回调方法中的权限处理；

 `HomeKit`、`HealthKit`、`Siri`需要开启`Capabilities`中的开关，即生成`projectName.entitlements`文件；

开源库[JLAuthorizationManager](https://github.com/123sunxiaolin/JLAuthorization)支持集成大部分常用的权限访问，便捷使用 **welcome to pull request or star**；

> 扫一扫下面的二维码，欢迎关注我的个人微信公众号攻城狮的动态（ID:iOSDevSkills），可在微信公众号进行留言，更多精彩技术文章，期待您的加入！一起讨论，一起成长！一起攻城狮！

