---
title: iOS开发中的这些权限，你搞懂了吗？ # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- iOS学习记录
tags: # 这里写的标签会自动汇集到 tags 页面上
- 系统权限
---

![懵懂也是一种美](http://upload-images.jianshu.io/upload_images/1713024-d2415c3dda2b7b65.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 写在前面

 APP开发避免不开系统权限的问题，如何在APP以更加友好的方式向用户展示系统权限，似乎也是开发过程中值得深思的一件事；
 
那如何提高APP获取iOS系统权限的通过率呢？有以下几种方式:

1.在用户打开APP时就向用户请求权限；

2.告知用户授权权限后能够获得好处之后，再向用户请求权限；

3.在绝对必要的情况下才向用户请求权限，例如：用户访问照片库时请求访问系统相册权限；

4.在展示系统权限的对话框前，先向用户显示自定义的对话框，若用户选择不允许，默认无操作，若用户选择允许，再展示系统对话框。

上述情况在开发过程中是经常遇到的，不同方式的选择会影响最后用户交互体验。这一点感悟正是源于上一周工作遇到的问题：适配iOS10，如何获取应用联网权限用以管理系统对话框的显示管理。当我把这个问题解决后，感觉有必要将常用的iOS系统权限做一个总结，以便后用。

#### 权限分类
 联网权限
 
 相册权限
 
 相机、麦克风权限
 
 定位权限
 
 推送权限
 
通讯录权限

 日历、备忘录权限

#### 联网权限


引入头文件 **@import CoreTelephony;**

 应用启动后，检测应用中是否有联网权限

      CTCellularData *cellularData = [[CTCellularData alloc]init];
      cellularData.cellularDataRestrictionDidUpdateNotifier =  ^(CTCellularDataRestrictedState state){
        //获取联网状态
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
        };
      };
      

查询应用是否有联网功能

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

#### 相册权限--iOS 9.0之前

导入头文件**@import AssetsLibrary;**

检查是否有相册权限

       ALAuthorizationStatus status = [ALAssetsLibrary authorizationStatus];
      switch (status) {
        case ALAuthorizationStatusAuthorized:
            NSLog(@"Authorized");
            break;
        case ALAuthorizationStatusDenied:
            NSLog(@"Denied");
            break;
        case ALAuthorizationStatusNotDetermined:
            NSLog(@"not Determined");
            break;
        case ALAuthorizationStatusRestricted:
            NSLog(@"Restricted");
            break;
            
        default:
            break;
      }
      
#### 相册权限--iOS 8.0之后

导入头文件**@import Photos;**

 
 检查是否有相册权限

      PHAuthorizationStatus photoAuthorStatus = [PHPhotoLibrary authorizationStatus];
      switch (photoAuthorStatus) {
        case PHAuthorizationStatusAuthorized:
            NSLog(@"Authorized");
            break;
        case PHAuthorizationStatusDenied:
            NSLog(@"Denied");
            break;
        case PHAuthorizationStatusNotDetermined:
            NSLog(@"not Determined");
            break;
        case PHAuthorizationStatusRestricted:
            NSLog(@"Restricted");
            break;
        default:
            break;
      }

获取用户权限：

 
       [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
        if (status == PHAuthorizationStatusAuthorized) {
            NSLog(@"Authorized");
        }else{
            NSLog(@"Denied or Restricted");
        }
        }];
        
#### 相机和麦克风权限

 导入头文件**@import AVFoundation;**

检查是否有相机或麦克风权限

      AVAuthorizationStatus AVstatus = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];//相机权限
      AVAuthorizationStatus AVstatus = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeAudio];//麦克风权限

      switch (AVstatus) {
        case AVAuthorizationStatusAuthorized:
            NSLog(@"Authorized");
            break;
        case AVAuthorizationStatusDenied:
            NSLog(@"Denied");
            break;
        case AVAuthorizationStatusNotDetermined:
            NSLog(@"not Determined");
            break;
        case AVAuthorizationStatusRestricted:
            NSLog(@"Restricted");
            break;
        default:
            break;
      }


获取相机或麦克风权限

      [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo completionHandler:^(BOOL granted) {//相机权限
        if (granted) {
            NSLog(@"Authorized");
        }else{
            NSLog(@"Denied or Restricted");
        }
      }];
    
      [AVCaptureDevice requestAccessForMediaType:AVMediaTypeAudio completionHandler:^(BOOL granted) {//麦克风权限
        if (granted) {
            NSLog(@"Authorized");
        }else{
            NSLog(@"Denied or Restricted");
        }
      }];

#### 定位权限

导入头文件**@import CoreLocation;**

由于iOS8.0之后定位方法的改变，需要在info.plist中进行配置；

![配置文件](http://upload-images.jianshu.io/upload_images/1713024-8b429a1fb5acd8a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 检查是否有定位权限

       BOOL isLocation = [CLLocationManager locationServicesEnabled];
      if (!isLocation) {
        NSLog(@"not turn on the location");
      }
      CLAuthorizationStatus CLstatus = [CLLocationManager authorizationStatus];
      switch (CLstatus) {
        case kCLAuthorizationStatusAuthorizedAlways:
            NSLog(@"Always Authorized");
            break;
        case kCLAuthorizationStatusAuthorizedWhenInUse:
            NSLog(@"AuthorizedWhenInUse");
            break;
        case kCLAuthorizationStatusDenied:
            NSLog(@"Denied");
            break;
        case kCLAuthorizationStatusNotDetermined:
            NSLog(@"not Determined");
            break;
        case kCLAuthorizationStatusRestricted:
            NSLog(@"Restricted");
            break;
        default:
            break;
      }

获取定位权限

      CLLocationManager *manager = [[CLLocationManager alloc] init];
      [manager requestAlwaysAuthorization];//一直获取定位信息
      [manager requestWhenInUseAuthorization];//使用的时候获取定位信息


在代理方法中查看权限是否改变

      - (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status{
       switch (status) {
        case kCLAuthorizationStatusAuthorizedAlways:
            NSLog(@"Always Authorized");
            break;
        case kCLAuthorizationStatusAuthorizedWhenInUse:
            NSLog(@"AuthorizedWhenInUse");
            break;
        case kCLAuthorizationStatusDenied:
            NSLog(@"Denied");
            break;
        case kCLAuthorizationStatusNotDetermined:
            NSLog(@"not Determined");
            break;
        case kCLAuthorizationStatusRestricted:
            NSLog(@"Restricted");
            break;
        default:
            break;
        }

      }

#### 推送权限


 检查是否有通讯权限

        UIUserNotificationSettings *settings = [[UIApplication sharedApplication] currentUserNotificationSettings];
      switch (settings.types) {
        case UIUserNotificationTypeNone:
            NSLog(@"None");
            break;
        case UIUserNotificationTypeAlert:
            NSLog(@"Alert Notification");
            break;
        case UIUserNotificationTypeBadge:
            NSLog(@"Badge Notification");
            break;
        case UIUserNotificationTypeSound:
            NSLog(@"sound Notification'");
            break;
            
        default:
            break;
      }

获取推送权限

      UIUserNotificationSettings *setting = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeSound | UIUserNotificationTypeAlert | UIUserNotificationTypeBadge categories:nil];
      [[UIApplication sharedApplication] registerUserNotificationSettings:setting];

#### 通讯录权限

导入头文件 **@import AddressBook;**

检查是否有通讯录权限

       ABAuthorizationStatus ABstatus = ABAddressBookGetAuthorizationStatus();
      switch (ABstatus) {
        case kABAuthorizationStatusAuthorized:
            NSLog(@"Authorized");
            break;
        case kABAuthorizationStatusDenied:
            NSLog(@"Denied'");
            break;
        case kABAuthorizationStatusNotDetermined:
            NSLog(@"not Determined");
            break;
        case kABAuthorizationStatusRestricted:
            NSLog(@"Restricted");
            break;
        default:
            break;
      }


获取通讯录权限

      ABAddressBookRef addressBook = ABAddressBookCreateWithOptions(NULL, NULL);
      ABAddressBookRequestAccessWithCompletion(addressBook, ^(bool granted, CFErrorRef error) {
        if (granted) {
            NSLog(@"Authorized");
            CFRelease(addressBook);
        }else{
            NSLog(@"Denied or Restricted");
        }
      });

#### 日历、备忘录权限

导入头文件

检查是否有日历或者备忘录权限

       typedef NS_ENUM(NSUInteger, EKEntityType) {
        EKEntityTypeEvent,//日历
        EKEntityTypeReminder //备忘
       };


      EKAuthorizationStatus EKstatus = [EKEventStore  authorizationStatusForEntityType:EKEntityTypeEvent];
      switch (EKstatus) {
        case EKAuthorizationStatusAuthorized:
            NSLog(@"Authorized");
            break;
        case EKAuthorizationStatusDenied:
            NSLog(@"Denied'");
            break;
        case EKAuthorizationStatusNotDetermined:
            NSLog(@"not Determined");
            break;
        case EKAuthorizationStatusRestricted:
            NSLog(@"Restricted");
            break;
        default:
            break;
      }


获取日历或备忘录权限

      EKEventStore *store = [[EKEventStore alloc]init];
      [store requestAccessToEntityType:EKEntityTypeEvent completion:^(BOOL granted, NSError * _Nullable error) {
        if (granted) {
            NSLog(@"Authorized");
        }else{
            NSLog(@"Denied or Restricted");
        }
      }];

#### 最后一点

素有获取权限的方法，多用于用户第一次操作应用，iOS 8.0之后，将这些设置都整合在一起，并且可以开启或关闭相应的权限。所有的权限都可以通过下面的方法打开：

      [[UIApplication sharedApplication] openURL:[NSURL URLWithString:UIApplicationOpenSettingsURLString]];

