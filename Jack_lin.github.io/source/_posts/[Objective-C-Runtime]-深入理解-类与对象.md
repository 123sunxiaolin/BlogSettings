---
title: Objective-C-Runtime：深入理解类与对象  #这是标题
categories:   # 这里写的分类会自动汇集到 categories 页面上，分类可以多级
- iOS学习记录
tags: # 这里写的标签会自动汇集到 tags 页面上
- Runtime
---

![在那樱花盛开的季节](https://upload-images.jianshu.io/upload_images/1713024-abc0e161c40d1866.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 概述
常说Objective-C是一门动态语言，那么问题来了，这个`动态`表现在那些方面呢？

其实最主要的表现就是**Objective-C**将很多静态语言在编译和链接时做的事情放到了运行时去做，它在运行时实现了对类、方法、成员变量、属性等信息的管理机制。

同时，运行时机制为我们开发过程提供很多便利之处，比如：
- 在运行时创建或者修改一个类；
- 在运行时修改成员变量、属性等；
- 在运行时进行消息分发和分发绑定；
......
与之对应实现的就是Objective-C的Runtime机制。

Objective-C的Runtime目前有两个版本：`Leagcy Runtime`和`Moden Runtime`。`Leagcy Runtime`是最早期给32位`Mac OX Apps`使用的，而`Moden Runtime`是给64位`Mac OX Apps`和`iOS Apps`使用的。

Runtime基本是C和汇编编写的，有一系列函数和数据结构组成的，具有公共接口的动态共享库，可见苹果为了动态系统的高效而作出的努力，你可以在[这里](http://www.opensource.apple.com/source/objc4/)下载到苹果维护的开源代码。

同时，GNU也有一个开源的Runtime版本,他们在努力保持一致。其头文件都存放在`/usr/include/objc`目录下。在[Objective-C Runtime Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html)中，有对Runtime函数使用细节的文档。

### 类与对象
#### 类的数据结构(Class)
类的数据结构可以在`objc/runtime.h`源码中找到，如下所示：

```
struct objc_class {
   //isa指针指向Class
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class  OBJC2_UNAVAILABLE; // 父类
    const char * _Nonnull name OBJC2_UNAVAILABLE; // 类名
    long version  OBJC2_UNAVAILABLE; // 类的版本信息，默认为0
    long info  OBJC2_UNAVAILABLE; // 类信息，供运行时使用的一些位标识
    long instance_size  OBJC2_UNAVAILABLE; // 类的实例变量大小
   // 类的成员变量列表
    struct objc_ivar_list * _Nullable ivars  OBJC2_UNAVAILABLE; 
    // 方法定义列表
    struct objc_method_list * _Nullable * _Nullable methodLists  OBJC2_UNAVAILABLE; 
    // 方法缓存
    struct objc_cache * _Nonnull cache  OBJC2_UNAVAILABLE; 
    // 协议列表
    struct objc_protocol_list * _Nullable protocols OBJC2_UNAVAILABLE; 
#endif

} OBJC2_UNAVAILABLE;
```

在Objective-C中`类`是由`Class`表示的，`Class`是一个指向`struct objc_class`的指针。
```
typedef struct objc_class *Class;
```
在这个类的数据结构中，有几个字段需要解释一下：

-  **isa**：在大多数的面向对象的语言中，都有**类**和**对象**的概念。其中，对象是类的实例，是通过类数据结构的定义创建出来的，对象的**isa**指针是指向其所属类的。同时，在Objective-C语言中，类本身也是一个对象，类作为对象时**isa**指针指向元类（Meta Class）,后面会详解；

- **super_class**：指向该类的父类，如果该类已经是根类（**NSObject** 或 **NSProxy**），则 其**super_class** 为**NULL**；

- **version**：该字段可以获取类的版本信息，在对象的序列化中可以通过类的版本信息来标识出不同版本的类定义中实例变量布局的改变。

#### objc_cache与cache
上文`object_class`中结构体中的`cache`字段，是用来缓存使用过的方法。这个字段是一个指向`objc_cache `的指针，具体数据结构如下所示：
```
struct objc_cache {
    unsigned int mask /* total = mask + 1 */                 OBJC2_UNAVAILABLE;
    unsigned int occupied                                    OBJC2_UNAVAILABLE;
    Method buckets[1]                                        OBJC2_UNAVAILABLE;
};
```
字段的具体描述如下：
- **mask**：整数类型，指定分配的缓存`bucket`的总数。在方法查找过程中，`Objective-C runtime`使用这个字段来确定开始线性查找数组的索引位置。指向方法`selector`的指针与该字段做一个`AND`位操作**(index = (mask & selector))**。这可以看作为一个简单的`hash`散列算法;
- `occupied`:一个整数，指定实际占用的缓存`bucket`的总数;
- `buckets`：指向`Method`数据结构指针的数组。这个数组可能包含不超过`mask+1`个元素。需要注意的是，指针可能是`NULL`，表示这个缓存`bucket`没有被占用。另外被占用的`bucket`可能是不连续的。这个数组可能会随着时间而增长。

关于上文`object_class`中结构体中的`cache`字段，对它的解释如下：

- **cache**：用于缓存最近使用的方法，一个对象可响应的方法列表中通常只有一部分是经常被调用的，**cache** 则是用来缓存最常调用的方法，从而避免每次方法调用时都去查找对象的整个方法列表，提升性能。
- 在一些结构较为复杂的类关系中，一个对象的响应方法可能来自于继承的类结构中，此情况下查找相应的响应方法时就会比较耗时，通常使用**cache**缓存可以减低查找时间；

举个栗子：
```
NSDictionary *dic= [[NSDictionary alloc] init];//执行过程
```
其缓存调用方法的流程：

- 1、`[NSDictionary alloc]`先被执行。由于`NSDictionary`没有`+alloc`方法，于是去父类`NSObject`中去查找；

- 2、 检测`NSObject`是否响应`+alloc`方法，发现响应，于是检测`NSDictionary`类，并根据其所需的内存空间大小开始分配内存空间，然后把`isa`指针指向`NSDictionary`类。同时，`+alloc`方法也被加进对应类的`cache`列表里;
- 3、执行`-init`方法，如果`NSDictionary`响应该方法，则直接将该方法加入`cache`列表；如果不响应，则去父类查找；
- 4、 在后期的操作中，如果再以`[[NSDictionary alloc] init]`这种方式来创建数组，则会直接从`cache`中取出相应的方法，直接调用。

#### 元类(Meta Class)
上面讲到，有时候类也是一个对象，这种类对象是某一种类的实例，这种类就是元类(Meta Class)。

好比类与对应的实例描述一样，元类则是类作为对象的描述。元类中方法列表对应的是类方法(Class Method)列表，这正是类作为一个对象所需要的。

当调用该方法`[NSArray alloc]`时，Runtime就会在对应的元类方法列表查找其类对应的方法，并匹配调用。

```
Meta Class是类对象的类。
```

官方的解释如下所示：
>Since a class is an object, it must be an instance of some other class: a metaclass. The metaclass is the description of the class object, just like the class is the description of ordinary instances. Class methods are described by the metaclass on behalf of the class object, just like instance methods are described by the class on behalf of the instance objects.

至此，又有了新的疑问：元类又是谁的实例呢？它的isa又指向谁呢？答案如下图所示：

![Class vs. Meta Class](https://upload-images.jianshu.io/upload_images/1713024-a6d34a8162448269.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由上图可以看出，元类的isa都指向根元类(Root Meta Class),即元类都是根元类的实例。

而根元类(Root Meta Class)的isa则指向自己，这样就不会无休止的关联下去了。

图中同样展示类和元类的继承关系，非常清晰易懂。

#### 类的实例数据结构
 在 Objective-C 中类的实例的数据结构是定义在` struct objc_object` 中(objc/objc.h):
```
/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};
```
可以看出，这个结构体只有一个字段，即指向该实例所属类的`isa`指针。

这个指针跟上面介绍的类的`isa`不一样：类的`isa`指向对应的元类(`Meta Class`)，实例的`isa`则是指向对应的类(`Class`)，而这个`Class`里包含上述所讲的数据：父类、类名、方法列表等等。

当我们向一个类的实例发送消息时，`Runtime`会根据实例对象的`isa`找到这个实例对象所属的类，然后再在这个类的方法列表和其父类的方法列表中查找与消息相对应的`selector`指向的方法，进而执行目标方法。

当创建某一个类的实例时，分配的内存中会包含一个`objc_object`数据结构，然后是类的实例变量的相关数据。

`NSObject`类的`alloc`和`allocWithZone:`方法是使用函数`class_createInstance`来创建`objc_object`数据结构。

我们常见的`id`是一个`struct objc_object`类型的指针。`id `类型的对象可以转换为任何一种类型的对象，它的作用有点类似 C 语言中的 `void *` 指针类型。
```
/// A pointer to an instance of a class.
typedef struct objc_object *id;
```
#### 相关函数

Objective-C的`Runtime`我们提供了很多运行时状态跟类与对象相关的函数。类的操作方法大部分是以`class_` 为前缀的，而对象的操作方法大部分是以`objc_`或`object_`为前缀，具体以分类的形式进行讨论。

##### 类相关函数
类的相关函数大部分是与`objc_class `结构体各个字段相关的方法。

###### 类名
```
// 获取类的类名
const char * class_getName ( Class cls );
```
- 如果`cls`传入`nil`时，则返回`nil`字符串；

######  父类(super_class)和元类(meta_class)

```
// 获取类的父类
Class class_getSuperclass ( Class cls );
// 判断给定的Class是否是一个元类
BOOL class_isMetaClass ( Class cls );
```
- `class_getSuperclass`函数，当`cls`为`Nil`或者`cls`为根类时，返回`Nil`。我们可以使用`NSObject`类的`superclass`方法来达到同样的目的;

- `class_isMetaClass`函数，如果是`cls`是元类，则返回YES；如果否或者传入的`cls`为`Nil`，则返回`NO`。

######  实例变量大小(instance_size)

```
// 获取实例大小
size_t class_getInstanceSize ( Class cls );
```
###### 成员变量(ivars)及属性
在`objc_class`中，所有的成员变量、属性的信息是放在链表`ivars`中的。`ivars`是一个数组，数组中每个元素是指向`Ivar`(变量信息)的指针。

1、成员变量操作函数：
```
// 获取类中指定名称实例成员变量的信息
Ivar class_getInstanceVariable ( Class cls, const char *name );
// 获取类成员变量的信息
Ivar class_getClassVariable ( Class cls, const char *name );
// 添加成员变量
BOOL class_addIvar ( Class cls, const char *name, size_t size, uint8_t alignment, const char *types );
// 获取整个成员变量列表
Ivar * class_copyIvarList ( Class cls, unsigned int *outCount );
```
- `class_getInstanceVariable`函数，它返回一个指向包含`name`指定的成员变量信息的`objc_ivar`结构体的指针(`Ivar`);
- `class_getClassVariable`函数，目前没有找到关于`Objective-C`中类变量的信息，一般认为`Objective-C`不支持类变量。注意，返回的列表不包含父类的成员变量和属性;

- `Objective-C`不支持往已存在的类中添加实例变量，因此不管是系统库提供的类，还是我们自定义的类，都无法动态添加成员变量；

- 当通过运行时来创建一个类的时候，我们就可以使用`class_addIvar`函数。不过需要注意的是，这个方法只能在`objc_allocateClassPair`函数与`objc_registerClassPair`之间调用。

- 另外，这个类也不能是元类。成员变量的按字节最小对齐量是`1<<alignment`。这取决于ivar的类型和机器的架构。如果变量的类型是指针类型，则传递`log2(sizeof(pointer_type))`;

- `class_copyIvarList`函数，它返回一个指向成员变量信息的数组，数组中每个元素是指向该成员变量信息的`objc_ivar`结构体的指针。这个数组不包含在父类中声明的变量。`outCount`指针返回数组的大小。需要注意的是，我们必须使用free()来释放这个数组。

2、属性相关的操作函数：
```
// 获取指定的属性
objc_property_t class_getProperty ( Class cls, const char *name );
// 获取属性列表
objc_property_t * class_copyPropertyList ( Class cls, unsigned int *outCount );
// 为类添加属性
BOOL class_addProperty ( Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount );
// 替换类的属性
void class_replaceProperty ( Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount );
```

3.`MAC OS X`系统支持使用垃圾回收器，`Runtime`提供了几个函数来确定一个对象的内存区域是否可以被垃圾回收器扫描，以处理`strong/weak`引用。这几个函数定义如下:
```
const uint8_t * class_getIvarLayout ( Class cls );
void class_setIvarLayout ( Class cls, const uint8_t *layout );
const uint8_t * class_getWeakIvarLayout ( Class cls );
void class_setWeakIvarLayout ( Class cls, const uint8_t *layout );

```
通常情况下，我们不需要去主动调用这些方法；在调用`objc_registerClassPair`时，会生成合理的布局。

###### 方法(methodLists)
```
// 添加方法
BOOL class_addMethod ( Class cls, SEL name, IMP imp, const char *types );
// 获取实例方法
Method class_getInstanceMethod ( Class cls, SEL name );
// 获取类方法
Method class_getClassMethod ( Class cls, SEL name );
// 获取所有方法的数组
Method * class_copyMethodList ( Class cls, unsigned int *outCount );
// 替代方法的实现
IMP class_replaceMethod ( Class cls, SEL name, IMP imp, const char *types );
// 返回方法的具体实现
IMP class_getMethodImplementation ( Class cls, SEL name );
IMP class_getMethodImplementation_stret ( Class cls, SEL name );
// 类实例是否响应指定的selector
BOOL class_respondsToSelector ( Class cls, SEL sel );
```

- `class_addMethod`的实现会覆盖父类的方法实现，但不会取代本类中已存在的实现，如果本类中包含一个同名的实现，则函数会返回NO。

- 如果要修改已存在实现，可以使用`method_setImplementation`。一个`Objective-C`方法是一个简单的`C`函数，它至少包含两个参数`self`和`_cmd`。所以，我们的实现函数(`IMP`参数指向的函数)至少需要两个参数，如下所示：

```
void myMethodIMP(id self, SEL _cmd)
{
    // implementation ....
}
```
与成员变量不同的是，我们可以为类动态添加方法，不管这个类是否已存在。

参数`types`是一个描述传递给方法的参数类型的字符数组，这就涉及到类型编码。

-  `class_getInstanceMethod`、`class_getClassMethod`函数，与`class_copyMethodList`不同的是，这两个函数都会去搜索父类的实现;

- `class_copyMethodLis`t函数，返回包含所有实例方法的数组。如果需要获取类方法，则可以使用`class_copyMethodList(object_getClass(cls), &count)`(一个类的实例方法是定义在元类里面)。**该列表不包含父类实现的方法**。`outCount`参数返回方法的个数，在获取到方法列表后，我们需要使用`free()`方法来释放它;

- `class_replaceMethod`函数，该函数的行为可以分为两种：如果类中不存在`name`指定的方法，则类似于`class_addMethod`函数一样会添加方法；如果类中已存在`name`指定的方法，则类似于`method_setImplementation`一样去替代原方法的实现；
- `class_getMethodImplementation`函数，该函数在向类实例发送消息时会被调用，并返回一个指向方法实现函数的指针。

- 这个函数会比`method_getImplementation(class_getInstanceMethod(cls, name))`更快。返回的函数指针可能是一个指向`runtime`内部的函数，而不一定是方法的实现。

- 例如，如果类实例无法响应`selector`，则返回的函数指针将是运行时消息转发机制的一部分;
- `class_respondsToSelector`函数，我们通常使用`NSObject`类的`respondsToSelector:`或者`instancesRespondToSelector:`方法来达到相同目的。

###### 协议(objc_protocol_list)

```
// 添加协议
BOOL class_addProtocol ( Class cls, Protocol *protocol );
// 返回类是否实现指定的协议
BOOL class_conformsToProtocol ( Class cls, Protocol *protocol );
// 返回类实现的协议列表
Protocol * class_copyProtocolList ( Class cls, unsigned int *outCount );
```
- `class_conformsToProtocol`函数可以使用`NSObject`类的`conformsToProtocol:`方法来替代;
- `class_copyProtocolList`函数返回的是一个数组，在使用后我们需要使用`free()`手动释放。

###### 版本(version)

```
// 获取版本号
int class_getVersion ( Class cls );
// 设置版本号
void class_setVersion ( Class cls, int version );

```

###### 其它

`runtime`还提供了两个不直接使用的函数来供`CoreFoundation`的`tool-free bridging`使用，即：
```
Class objc_getFutureClass ( const char *name );
void objc_setFutureClass ( Class cls, const char *name );

```

使用上述函数时，需要特别的注意一下细节信息和使用规范，具体可以查阅 [Objective-C Runtime Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html)。

##### 动态创建类与对象
Runtime提供在运行时创建类与对象的方法。
###### 动态创建类
```
// 创建一个新类和元类
Class objc_allocateClassPair ( Class superclass, const char *name, size_t extraBytes );
// 销毁一个类及其相关联的类
void objc_disposeClassPair ( Class cls );
// 在应用中注册由objc_allocateClassPair创建的类
void objc_registerClassPair ( Class cls );
```
- `objc_allocateClassPair`函数：如果我们要创建一个根类，则`superclass`指定为`Nil`。`extraBytes`通常指定为0，该参数是分配给类和元类对象尾部的索引`ivars`的字节数;
- 创建一个新类，首先，我们需要调用`objc_allocateClassPair`。然后使用诸如`class_addMethod`、`class_addIvar`等函数来为新创建的类添加方法、实例变量和属性等。

- 完成这些后，我们需要调用`objc_registerClassPai`r函数来注册类，之后这个新类就可以在程序中使用了;
- 实例方法和实例变量应该添加到类自身上，而类方法应该添加到类的元类上;
- `objc_disposeClassPair`函数用于销毁一个类，不过需要注意的是，如果程序运行中还存在类或其子类的实例，则不能调用针对类调用该方法，在后面的栗子中也有该方面的讲解。

###### 动态创建对象
```
// 创建类实例
id class_createInstance ( Class cls, size_t extraBytes );
// 在指定位置创建类实例
id objc_constructInstance ( Class cls, void *bytes );
// 销毁类实例
void * objc_destructInstance ( id obj );
```
- `class_createInstance`函数：创建实例时，会在默认的内存区域为类分配内存。
- `extraBytes`参数表示分配的额外字节数。这些额外的字节可用于存储在类定义中所定义的实例变量之外的实例变量，该函数在ARC环境下无法使用 ；
- 调用`class_createInstance`的效果与`+alloc`方法类似。不过在使用`class_createInstance`时，我们需要确切的知道我们要用它来做什么。在下面的例子中，我们用NSString来测试一下该函数的实际效果:
```
- (void)testInstanceMethod{
    id theObject = class_createInstance([NSString class], sizeof(unsigned));
    id str1 = [theObject init];
    NSLog(@"%@", [str1 class]);
    
    id str2 = [[NSString alloc] initWithString:@"test"];
    NSLog(@"%@", [str2 class]);
}
```
输出结果：
```
2018-03-21 22:55:18.503665+0800 RuntimeUsage[2774:32008] NSString
2018-03-21 22:55:21.624606+0800 RuntimeUsage[2774:32008] __NSCFConstantString
```

- 可以看到，使用`class_createInstance`函数获取的是`NSString`实例，而不是类簇中的默认占位符类`__NSCFConstantString`;
- `objc_constructInstance`函数：在指定的位置(`bytes`)创建类实例;
- `objc_destructInstance`函数：销毁一个类的实例，但不会释放并移除任何与其相关的引用;

##### 实例操作函数
实例操作函数主要是针对我们创建的实例对象的一系列操作函数。

我们可以使用这组函数来从实例对象中获取我们想要的一些信息，如实例对象中变量的值。这组函数可以分为三小类：

1、针对整个对象进行操作的函数，这类函数包含：
```
// 返回指定对象的一份拷贝
id object_copy ( id obj, size_t size );
// 释放指定对象占用的内存
id object_dispose ( id obj );
```
有这样一种场景，假设我们有类A和类B，且类B是类A的子类。

类B通过添加一些额外的属性来扩展类A。现在我们创建了一个A类的实例对象，并希望在运行时将这个对象转换为B类的实例对象，这样可以添加数据到B类的属性中。

这种情况下，我们没有办法直接转换，因为B类的实例会比A类的实例更大，没有足够的空间来放置对象。此时，我们就要以使用以上几个函数来处理这种情况，如下代码所示：
```
NSObject *a = [[NSObject alloc] init];
id newB = object_copy(a, class_getInstanceSize(MyClass.class));
object_setClass(newB, MyClass.class);
object_dispose(a);
```
2、针对对象实例变量进行操作的函数，这类函数包含：
```
 修改类实例的实例变量的值
Ivar object_setInstanceVariable ( id obj, const char *name, void *value );
// 获取对象实例变量的值
Ivar object_getInstanceVariable ( id obj, const char *name, void **outValue );
// 返回指向给定对象分配的任何额外字节的指针
void * object_getIndexedIvars ( id obj );
// 返回对象中实例变量的值
id object_getIvar ( id obj, Ivar ivar );
// 设置对象中实例变量的值
void object_setIvar ( id obj, Ivar ivar, id value );
```
如果实例变量的`Ivar`已经知道，那么调用`object_getIvar`会比`object_getInstanceVariable`函数快，相同情况下，`object_setIvar`也比`object_setInstanceVariable`快。

3、针对对象的类进行操作的函数，这类函数包含：
```
// 返回给定对象的类名
const char * object_getClassName ( id obj );
// 返回对象的类
Class object_getClass ( id obj );
// 设置对象的类
Class object_setClass ( id obj, Class cls );

```
##### 获取类定义
`Objective-C`动态运行库会自动注册我们代码中定义的所有的类。

我们也可以在运行时创建类定义并使用`objc_addClass`函数来注册它们。`Runtime`提供了一系列函数来获取类定义相关的信息，这些函数主要包括：
```
// 获取已注册的类定义的列表
int objc_getClassList ( Class *buffer, int bufferCount );
// 创建并返回一个指向所有已注册类的指针列表
Class * objc_copyClassList ( unsigned int *outCount );
// 返回指定类的类定义
Class objc_lookUpClass ( const char *name );
Class objc_getClass ( const char *name );
Class objc_getRequiredClass ( const char *name );
// 返回指定类的元类
Class objc_getMetaClass ( const char *name );
```
- `objc_getClassList`函数：获取已注册的类定义的列表。我们不能假设从该函数中获取的类对象是继承自`NSObject`体系的，所以在这些类上调用方法是，都应该先检测一下这个方法是否在这个类中实现。

下面代码演示了该函数的用法:

```
- (void)testGetNumberClass{
    int numClasses;
    Class *classes = NULL;
    
    numClasses = objc_getClassList(NULL, 0);
    if (numClasses > 0) {
        classes = (Class *) malloc(sizeof(Class) * numClasses);
        numClasses = objc_getClassList(classes, numClasses);
        NSLog(@"count of classes:%d", numClasses);
        
        for (int i = 0; i < numClasses; i ++) {
            Class cls = classes[I];
           NSLog(@"class name: %s", class_getName(cls));
        }
    }
    free(classes);
}
```
输出的结果：
```
2018-03-21 23:58:57 RuntimeUsage[3378:65534] count of classes:11809
2018-03-21 23:59 RuntimeUsage[3378:65534] class name: _CNZombie_
2018-03-21 23:59 RuntimeUsage[3378:65534] class name: JSExport
2018-03-21 23:59 RuntimeUsage[3378:65534] class name: NSLeafProxy
2018-03-21 23:59 RuntimeUsage[3378:65534] class name: NSProxy
2018-03-21 23:59 RuntimeUsage[3378:65534] class name: _UITargetedProxy
2018-03-21 23:59 RuntimeUsage[3378:65534] class name: _UIViewServiceReplyControlTrampoline
2018-03-21 23:59:04  RuntimeUsage[3378:65534] class name: _UIViewServiceReplyAwaitingTrampoline
····  还有很多
```
- 获取类定义的方法有三个：`objc_lookUpClass`、`objc_getClass`和`objc_getRequiredClass`。

- 如果类在运行时未注册，则`objc_lookUpClass`会返回`nil`，而`objc_getClass`会调用类处理回调，并再次确认类是否注册，如果确认未注册，再返回`nil`。
- 
- 而`objc_getRequiredClass`函数的操作与`objc_getClass`相同，只不过如果没有找到类，则会杀死进程。
- `objc_getMetaClass`函数：如果指定的类没有注册，则该函数会调用类处理回调，并再次确认类是否注册，如果确认未注册，再返回nil。不过，每个类定义都必须有一个有效的元类定义，所以这个函数总是会返回一个元类定义，不管它是否有效。

#### 运行时操作操作类与对象的示例代码
- #####实例、类、父类、元类关系结构的示例代码

首先，创建继承关系为`Animal->Dog->NSObject`的几个类，然后使用Runtime的方法打印其中的关系，运行结果如下所示：

```
- (void)verifyClassTypeRelation{
    
    //Use `object_getClass` get Class's `isa`
    
    Dog *aDog = [[Dog alloc] init];
    Class dogCls = object_getClass(aDog);
    NSLog(@"isa->%@ , super_class->%@", dogCls, class_getSuperclass(dogCls));
    // print:isa->Dog, super_class->Animal
    Class dogMetaCls = objc_getMetaClass([NSStringFromClass(dogCls) UTF8String]);
    if (class_isMetaClass(dogMetaCls)) {
        NSLog(@"YES, metaCls->%@ , metaCls's super_Class->%@, metaCls's isa->%@", dogMetaCls, class_getSuperclass(dogMetaCls), object_getClass(dogMetaCls));
        //print: YES, metaCls->Dog , metaCls's super_Class->Animal, metaCls's isa->NSObject
    }else{
        NSLog(@"NO");
    }
    
    Animal *animal = [[Animal alloc] init];
    Class animalCls = object_getClass(animal);
    NSLog(@"isa->%@ , super_class->%@", animalCls, class_getSuperclass(animalCls));
    //print: isa->Animal , super_class->NSObject
    Class animalMetaCls = objc_getMetaClass([NSStringFromClass(animalCls) UTF8String]);
    if (class_isMetaClass(animalMetaCls)) {
        NSLog(@"YES, metaCls->%@ , metaCls's super_Class->%@, metaCls's isa->%@", animalMetaCls, class_getSuperclass(animalMetaCls), object_getClass(animalMetaCls));
        //print:YES, metaCls->Animal , metaCls's super_Class->NSObject, metaCls's isa->NSObject
    }else{
        NSLog(@"NO");
    }
    
    Class viewMetaCls = objc_getMetaClass([NSStringFromClass([UIView class]) UTF8String]);
    if (class_isMetaClass(viewMetaCls)) {
        NSLog(@"YES, metaCls->%@ , metaCls's super_Class->%@, metaCls's isa->%@", viewMetaCls, class_getSuperclass(viewMetaCls), object_getClass(viewMetaCls));
        //print:YES, metaCls->UIView , metaCls's super_Class->UIResponder, metaCls's isa->NSObject
    }
    
    Class rootMetaCls = objc_getMetaClass([NSStringFromClass([NSObject class]) UTF8String]);
    if (class_isMetaClass(rootMetaCls)) {
        NSLog(@"YES, metaCls->%@ , metaCls's super_Class->%@, metaCls's isa->%@", rootMetaCls, class_getSuperclass(rootMetaCls), object_getClass(rootMetaCls));
        //print:YES, metaCls->NSObject , metaCls's super_Class->NSObject, metaCls's isa->NSObject
    }
    
}
```
打印信息如下所示:
```
isa->Dog, super_class->Animal
YES, metaCls->Dog , metaCls's super_Class->Animal, metaCls's isa->NSObject
isa->Animal , super_class->NSObject
YES, metaCls->Animal , metaCls's super_Class->NSObject, metaCls's isa->NSObject
YES, metaCls->UIView , metaCls's super_Class->UIResponder, metaCls's isa->NSObject
YES, metaCls->NSObject , metaCls's super_Class->NSObject, metaCls's isa->NSObject
```

需要特别注意一下，`Object_getClass`可以获取当前对象的`isa`。以`Dog`类打印信息为例，解释一下具体实现的原理：
```
isa->Dog, super_class->Animal
YES, metaCls->Dog , metaCls's super_Class->Animal, metaCls's isa->NSObject
```
- 首先，通过`Object_getClass `获取实例`aDog`的Class(isa)为`Dog`;
- 然后，通过`class_getSuperclass `获取`Dog`的父类为`Animal`类；
- 通过`objc_getMetaClass `指定类名，获取对应的元类，通过`class_isMetaClass `方法可以判断一个类是否为指定类的元类，这里确认后，打印出`YES`,打印出的元类名称为`Dog `;打印元类父类为`Animal`;在通过
`Object_getClass `获取元类的isa，指向`NSObject`。

同理可得，`Animal`和`UIView`打印信息解释同上。`NSobject`,它元类的isa指针还是指向自己的类——`NSobject`。打印的信息与上述的关系图保持一致。

#### 动态操作类与实例的代码

动态创建类的源码
```
/***********************************************************************
* _objc_allocateFutureClass
* Allocate an unresolved future class for the given class name.
* Returns any existing allocation if one was already made.
* Assumes the named class doesn't exist yet.
* Locking: acquires runtimeLock
**********************************************************************/
Class _objc_allocateFutureClass(const char *name)
{
    rwlock_writer_t lock(runtimeLock);

    Class cls;
    NXMapTable *map = futureNamedClasses();

    if ((cls = (Class)NXMapGet(map, name))) {
        // Already have a future class for this name.
        return cls;
    }

    cls = _calloc_class(sizeof(objc_class));
    addFutureNamedClass(name, cls);

    return cls;
}
```

```
- (void)dynamicAddMethod{
    
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wundeclared-selector"
    
    //1、Create and register class, add method to class
    Class cls = objc_allocateClassPair([Animal class], "Lion", 0);// 当为`Cat`时，返回的创建类cat地址为0x0,将`Cat`作为关键字
    //method: 返回`int32_t`,type使用`i`;参数：`id self`,type使用`@`;`SEL _cmd`,type使用`:`;
    //`NSDictionary *dic`,type使用`@`.综上，type使用'i@:@'
    ///具体类型可参照 https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html
    BOOL isAddSuccess = class_addMethod(cls, @selector(howlMethod), (IMP)testRuntimeMethodIMP, "i@:@");
    NSLog(@"%@", isAddSuccess ? @"添加方法成功" : @"添加方法失败");
    //You can only register a class once.
    objc_registerClassPair(cls);
    
    //2、Create instance of class
    id whiteCat = [[cls alloc] init];
     NSLog(@"%@, %@", object_getClass(whiteCat), class_getSuperclass(object_getClass(whiteCat)));
    // Print: Lion, Animal
    Class whiteCatCls = object_getClass(whiteCat);
    Class metaCls = objc_getMetaClass([NSStringFromClass(whiteCatCls) UTF8String]);
    if (class_isMetaClass(metaCls)) {
        NSLog(@"YES, %@, %@, %@", metaCls, class_getSuperclass(metaCls), object_getClass(metaCls));
        // Print: YES, Lion, Animal, NSObject
    }else{
        NSLog(@"NO");
    }
    
    //3、Method of class
    unsigned int methodCount = 0;
    Method *methods = class_copyMethodList(cls, &methodCount);
    for (int32_t i = 0; i < methodCount; i ++) {
        Method aMethod = methods[I];
        NSLog(@"%@, %s", NSStringFromSelector(method_getName(aMethod)), method_getTypeEncoding(aMethod));
        //print:howlMethod, I@:@
    }
    free(methods);
    
    //4、call method
    int32_t result = (int)[whiteCat performSelector:@selector(howlMethod) withObject:@{@"name":@"lion", @"sex": @"male"}];
    NSLog(@"%d", result);//print:9
    
    //5、destory instance and class
    whiteCat = nil;
    
    // Do not call this function if instances of the cls class or any subclass exist.
    objc_disposeClassPair(cls);
    
#pragma clang diagnostic pop
}

```

打印的信息如下所示：
```
添加方法成功
 Lion, Animal
YES, Lion, Animal, NSObject
howlMethod, I@:@
testRuntimeMethodIMP: {
    name = lion;
    sex = male;
}
9
```
在执行`objc_allocateClassPair `中，类的名称设置为`Cat`时，创建出的`Class`的地址始终指向`0x0`,创建类失败。

猜测其中的原因可能是`Cat`与内部的关键字冲突了，导致类创建失败，改为`cat`或者其他的都可以创建成功；

- 在上面的代码中，在运行时动态创建了`Animal` 的一个子类：`Lion`；接着为这个类添加了方法和实现；
- 打印了 `Lion` 的类、父类、元类相关信息；
- 遍历和打印了 `Lion` 的方法的相关信息；
- 调用了 `Lion` 的方法；
- 最后销毁了实例和类。

针对上述代码，有几点需要特殊说明一下：
- 对于`#pragma clang diagnostic...`几行代码，是用于忽略编译器对于未声明@selector的警告信息的，在代码中，我们动态地为一个类添加方法，不会事先声明的；
-   `class_addMethod()` 函数的最后一个参数 `types` 是描述方法返回值和参数列表的字符串。

- 我们的代码中的用到的 `i@:@` 四个字符分别对应着：返回值 `int32_t`、参数` id self`、参数 `SEL _cmd`、参数 `NSDictionary *dic`。这个其实就是类型编码(*Type Encoding*)的概念。在 Objective-C 中，为了协助 Runtime 系统，编译器会将每个方法的返回值和参数列表编码为一个字符串，这个字符串会与方法对应的 selector 关联。更详细的知识可以查阅 [Type Encodings](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html);
- 使用 `objc_registerClassPair()` 函数需要注意，不能注册已经注册过的类;
- 使用` objc_disposeClassPair()` 函数时需要注意，**当一个类的实例和子类还存在时，不能去销毁一个类**，谨记；

##### isKindOf 和 isMemberOf
举个栗子：
```
@interface TestMetaClass: NSObject
@end

@implementation TestMetaClass
@end

int main(int argc, char * argv[]) {
    @autoreleasepool {
        BOOL result1 = [(id)[NSObject class] isMemberOfClass:[NSObject class]];
        BOOL result2 = [(id)[NSObject class] isKindOfClass:[NSObject class]];
        BOOL result3 = [(id)[TestMetaClass class] isMemberOfClass:[TestMetaClass class]];
        BOOL result4 = [(id)[TestMetaClass class] isKindOfClass:[TestMetaClass class]];
        NSLog(@"%d %d %d %d", result1, result2, result3, result4);
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

//log
2018-02-09 16:45:54.048040+0800 RuntimeUsage[9220:5754652] 0 1 0 0
```

关于**isMemberOfClass**和**isKindOfClass**在**Object.mm**中的实现，具体如下：
```
- (BOOL)isMemberOf:aClass
{
     return isa == (Class)aClass;
}

- (BOOL)isKindOf:aClass
{
     Class cls;
     for (cls = isa; cls; cls = cls->superclass)
          if (cls == (Class)aClass)
               return YES;
     return NO;
}
```
在result1中，从`isMemberOf`看出`NSObject class`的`isa`第一次会指向`NSObject`的`Meta Class`，因此`NSObject`的`Meta Class`与`NSObject class`是不相等，返回`FALSE`；

在result2中，`isKindOf`第一次指向`NSObject`的`Meta Class`，接着执行`superclass`时，根元类`NSObject`的`Meta Class`根据上面所讲的其`superclass`指针会闭环指向`NSObject class`，从而结果值为`TRUE`;

在result3中，`isa`会指向`TestMetaClass` 的`Meta Class`,与`TestMetaClass Class`不相等，结果值为`FALSE`; 

在result4中，第一次是`TestMetaClass Meta Class`,第二次`super class`后就是`NSObject Meta Class`，结果值为`FALSE`;

以上再次验证了，`NSObject Meta Class`的`isa`指针指向自身，其`super class`指向`NSObject`。

### 小结
本文着重讲解了在Runtime时类与对象相关方法和数据结构，通过这些讲解可以让大家对Objective-C底层类与对象实现有大致的了解，并且可以为大家平常编程过程提供一些思路上的启发。

测试使用的栗子(Demo)都在[篮子里](https://github.com/123sunxiaolin/RuntimeUsage.git)

### 参考
-  [Objective-C Runtime的数据类型](http://www.cnblogs.com/whyandinside/archive/2013/02/26/2933552.html)
-  [What's the difference between doing alloc and class_createInstance](https://stackoverflow.com/questions/3805499/whats-the-difference-between-doing-alloc-and-class-createinstance)

- [Objective-C Runtime Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)
