---
title: iOS中，系统相册的那些事 # 这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- iOS学习记录
tags: # 这里写的标签会自动汇集到 tags 页面上
- 系统相册
---


![拍照也是一门技术](http://upload-images.jianshu.io/upload_images/1713024-ebc88609d6c93956.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 写在前面

在手机APP日益增加的前提下，如何更好的提升用户的交互体验似乎成为衡量一个APP重要指标。上述的感悟源于实际工作的需求，就是在APP中添加一个更换用户头像的功能。

也许别人会认为这样一个小功能不算什么，但从用户交互角度考虑，这样一个功能的设计有一定学问，待我慢慢道来。

#### 获取相册最直接的方式——UIImagePickerController

功能介绍：可直接显示分组的相处的列表，用户选择不同相册的照片后，可在委托方法中获得该图片对象；

API提供三种数据源：

      UIImagePickerControllerSourceTypeCamera: //拍照
      UIImagePickerControllerSourceTypePhotoLibrary: //相册
      UIImagePickerControllerSourceTypeSavedPhotosAlbum: //图片库
      
基本使用

      //UIImagePickerController 属于UIKit
      UIImagePickerController *imagePicker = [[UIImagePickerController alloc] init];
      // 若设备支持相机，使用拍照功能；否则从照片库中选择
      if ([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {
       imagePicker.sourceType = UIImagePickerControllerSourceTypeCamera;
      } else {
       imagePicker.sourceType = UIImagePickerControllerSourceTypeSavedPhotosAlbum;
      }
      imagePicker.delegate = self; //设置委托，

 跳转到系统相册界面
        _imagePickerController.allowsEditing = YES;//允许拍照完对照片进行裁剪
        
        
        [self presentViewController:_imagePickerController animated:YES completion:nil];
写到这里，基本的调用系统相册的功能就实现了，唯一需要做的是参数配置

 遵守的协议
**UINavigationControllerDelegate**,**UIImagePickerControllerDelegate**

        代理方法
        - (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary<NSString *,id> *)info{
        //成功获取照片
        }

        - (void)imagePickerControllerDidCancel:(UIImagePickerController *)picker{
            //获取照片失败
        }

 捕捉多媒体的的类型 UIImagePickerControllerCameraCaptureMode
     
         UIImagePickerControllerCameraCaptureModePhoto,//照片
         UIImagePickerControllerCameraCaptureModeVideo//视频
   
         
         
 摄像头的类型 UIImagePickerControllerCameraDevice
 
          UIImagePickerControllerCameraDeviceRear,//后置摄像头
            UIImagePickerControllerCameraDeviceFront //前置摄像头  
                          
设置闪光灯的模式  UIImagePickerControllerCameraFlashMode 
 
        UIImagePickerControllerCameraFlashModeOff  = -1,//关闭闪光灯
           UIImagePickerControllerCameraFlashModeAuto = 0,//自动模式
        UIImagePickerControllerCameraFlashModeOn   = 1//开启闪光灯
        
#### 自定义相册方式之一 ALAssetsibrary

基本介绍：该框架可实现自定义相册，实现定制的图片选择器，可支持多选、自定义界面，只不过API在iOS9.0版本被标记废弃，即iOS9.0之前的版本可以使用ALAssetsLibrary实现自定义，iOS9.0之后的版本需要使用Photos.fraework。
 
 成员介绍：
 
       1.ALAssetsGroup:映射照片库（ALAssetsLibrary）中的一个相册，通过ALAssetsGroup可以获取相册相应的信息，以及获取到对应相册下的所有图片资源； 
                                                                
       2.ALAsset:对应相册中的一张图片或者一个视频，并且包含对应图片和视频的详细信息，可获取图片对应的缩略图，还可通过ALAsset的实例方法保存图片和视频；
      
       3.ALAssetRepresentation:可简单理解为对ALAsset的封装，对于给定的ALAsset都至少会对应一个ALAssetRepresentation，通过ALAsset的实例方法

       defaultRepresentation获得对应的ALAssetRepresentation，例如使用系统相机的拍摄的RAW＋JPEG照片，则会有两个ALAssetRepresentation，一个封装了RAW信息，另一个封装了JPEG的信息。通过ALAssetRepresentation可以获取ALAsset的原图、全屏图、文件名等信息；
      
自定义行相册的思路

        1.实例化照片库，获取所有的相册；
        2.展示相册中的所有照片，可自义展示样式，多以集合视图的形式展现；
        3.选择照片后返回上级界面或者进入预览图。
 
 具体实现
 
      1.导入头文件** #import <AssetsLibrary/ALAssetsLibrary.h>** 或者 ** @import AssetsLibrary;**
     
      2.实例化AssetsLibrary
      **ALAssetsLibrary *assertLibray = [[ALAssetsLibrary alloc]init];**
     
     3.**遍历照片库所有的相册**
     
         groups = [NSMutableArray array];
            //遍历相册
            [assetLibrary enumerateGroupsWithTypes:ALAssetsGroupSavedPhotos usingBlock:^(ALAssetsGroup *group, BOOL *stop) {
        if (group) {//遍历相册未结束
            //设置过滤器
            [group setAssetsFilter:[ALAssetsFilter allPhotos]];
            if (group.numberOfAssets) {
                [groups addObject:group];
            }
        }else{//遍历结束
            if (groups.count) {
                //当相册个数不为零时，开始遍历相册
                [self enumenumerateAssets];
            }else{
                NSLog(@"no group");
            }
            
        }
            } failureBlock:^(NSError *error) {
        if (error) {
            NSLog(@"error = %@", [error description]);
        }
        }];
        
    4.**遍历相册中的照片**
    
        - (void)enumerateAssets{
    
            NSMutableArray *assetArray = [NSMutableArray new];
          for (ALAssetsGroup *group in  groups) {
        //遍历所有的照片-方式一
        [group enumerateAssetsWithOptions:NSEnumerationReverse usingBlock:^(ALAsset *result, NSUInteger index, BOOL *stop) {
            if (result) {
                [assetArray addObject:result];
            }else{
                NSLog(@"here is no photos");
            }
        }];
        
        //遍历指定索引的照片-方式二
        NSInteger fromIndex = 0;
        NSInteger toIndex = 5;
        [group enumerateAssetsAtIndexes:[NSIndexSet indexSetWithIndex:toIndex] options:NSEnumerationReverse usingBlock:^(ALAsset *result, NSUInteger index, BOOL *stop) {
            if (index > toIndex) {
                *stop = YES;//遍历到最后一张，停止遍历
            }else{
                if (result) {
                    [assetArray addObject:result];//存储照片
                }else{
                    NSLog(@"enumeration has ended!");
                }
            }
        }];
        }}
        
    5 **完成上述步骤后，就能获得所有相册和相册中对应的所有照片，接下来就可以根据自己的需求自定义显示界面了，这里就不再一一赘述了。**
 
#### 自定义相册方式之二Photos.framework

 基本介绍：Photos是苹果在iOS8.0提出的API，是目前，苹果推荐的照片框架，学习一下还是很有必要的；
 
主要成员介绍：

      1.PHAsset：代表照片库中的一个资源，与ALAsset类似，通过PHAsset可以获取和保存资源；
      2.PHFetchOptions：获取资源时的参数；
      3.PHAssetCollection：PHCollection的子类，表示一个相册或者一个时刻，也可以是一个【智能相册】（系统提供的一系列相册集合，包括最近删除、相机相册、最爱相册等等）中的一个；
      4.PHFetchResult：表示一系列资源结果的集合，也可以是相册资源集合，一般情况下，可以从PHCollection或PHAsset的类方法中获取；
      5.PHImageManager：用于处理资源的加载，图片加载的过程带有缓存处理；
      6.PHImageRequestOptions：控制加载资源的时一系列参数。
      
 具体使用

       1.导入框架**@import Photos;**
          
       2.**获取系统相册,系统提供下列三种获取不同分类相册的方法。**
        
        //获得所有智能相册
        PHFetchResult *smartAlbums = [PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeSmartAlbum subtype:PHAssetCollectionSubtypeAlbumRegular options:nil];
        
        //获取所有用户创建的相册
      PHFetchResult *topLevelUserCollections = [PHCollectionList fetchTopLevelUserCollectionsWithOptions:nil];
     
      //获取所有资源的集合，并按资源的创建时间排序
      PHFetchOptions *allPhotoOptions = [[PHFetchOptions alloc] init];
      allPhotoOptions.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"creationDate" ascending:YES]];
      PHFetchResult *allphotos = [PHAsset fetchAssetsWithOptions:allPhotoOptions];
      
    3 获取对应的照片资源
    
         //列出所有智能相册，此时smartAlbums保存是各个智能相册对应的PHAssetCollection
        PHFetchResult *smartAlbums = [PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeSmartAlbum subtype:PHAssetCollectionSubtypeAlbumRegular options:nil];
    
         for (NSInteger i = 0; i < smartAlbums.count; i ++) {
        //从中获取一相册
        PHCollection *collection = smartAlbums[i];
        if ([collection isKindOfClass:[PHAssetCollection class]]) {
            //判断是否是PHAssetCollection类
            PHAssetCollection *assetCollection = (PHAssetCollection *)collection;
            //从每个智能相册中获取资源集合,可以看做是PHAsset的集合
            PHFetchResult *photoSet = [PHAsset fetchAssetsInAssetCollection:assetCollection options:nil];
           for (NSInteger j = 0; j < photoSet.count; i ++) {
                //获取其中一个资源
                PHAsset *asset = photoSet[i];
            }     
            }else{
            NSLog(@"not PHAssetCollection");
        }
      }
      
      //获取所有资源的集合，并按资源的创建时间排序
      PHFetchOptions *allPhotoOptions = [[PHFetchOptions alloc] init];
      allPhotoOptions.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"creationDate" ascending:YES]];
      PHFetchResult *allphotos = [PHAsset fetchAssetsWithOptions:allPhotoOptions];
      for (NSInteger i = 0; i < allphotos.count ; i ++) {
        //获取一个资源
        PHAsset *asset = allphotos[i];
      }
      
     4 显示图片资源，需要用到PHImageManager类
     
       PHImageManager *imageManager = [[PHImageManager alloc]init];
      //获取第一张照片资源
      PHAsset *asset = allphotos[0];
      //设定显示照片的尺寸
      CGFloat width = _showImageView.frame.size.width;
      CGFloat height = _showImageView.frame.size.height;
      CGSize imageSize = CGSizeMake(width, height);
    
      [imageManager requestImageForAsset:asset
                            targetSize:imageSize
                           contentMode:PHImageContentModeAspectFill
                               options:nil
                         resultHandler:^(UIImage * _Nullable result, NSDictionary * _Nullable info) {
                             //显示照片
                             _showImageView.image = result;
                         }];
                         
#### ALAssetsibrary与Photos的对比

**适用的iOS版本不同**，ALAssetsibrary适用于iOS9.0之前，Photos适用于iOS9.0之后；

**获取资源的方式不同：**ALAssetsibrary都是以**枚举**的方式获取资源的，遍历照片库（ALAssetsibrary）获得相册（ALAssetsGroup），通过遍历相册获得具体资源（ALAsset），枚举方式获取资源，存在效率低且不灵活的缺点；Photos采用**拉取**的方式获取资源；

由上述方法可知，多使用**PHFetchResult**获取对应资源，不采用枚举方式获取资源，在效率上会有所提高；

>以上内容均来自工作学习中的心得，有不足的地方欢迎大家前来讨论，共同提高。
