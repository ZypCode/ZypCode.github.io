---
title: OC hook
date: 2019-01-22 17:25:57
sage: true
categories:
- Objective-C
tags:
- Objective-C
- 技术分享
---
对于项目中使用到的Objective-c注入技术，借鉴@丁捷的分享进行自己学习该技术的总结。
## 一、原理
注入的原理就是在被注入程序启动前，使用自己实现的方法替换相应方法，而当程序进行特定操作时即会执行自己实现的方法，达到目的。

>**Selector** (typedef struct objc_selector *SEL)

Selector是一个运行时被注册（或映射）、代表一个方法的C类型字符串，由编译器产生并且当类被加载进内存时由运行时自动进行名字和实现的映射。

>**Method**(typedef struct objc_method *Method)

Method是一个不透明的用来代表一个方法的定义的类型。

>**Implementation**(typedef id (*IMP)(id, SEL,...))

Implementation指向一个方法的实现的最开始的地方，该方法的第一个参数指向调用方法的自身（即内存中类的实例对象，若是调用类方法，该指针则是指向元类对象metaclass）。第二个参数是这个方法的名字selector，该方法的真正参数紧随其后。



>在运行时，类（Class）维护了一个消息分发列表来解决消息的正确发送。每一个消息列表的入口是一个方法（Method），这个方法映射了一对键值对，其中键值是这个方法的名字 selector（SEL），值是指向这个方法实现的函数指针 implementation（IMP）。 Method swizzling 修改了类的消息分发列表使得已经存在的 selector 映射了另一个实现 implementation，同时重命名了原生方法的实现为一个新的 selector。



*示例：*

1. 为NSArray添加类别NSArray (Swizzle)，新增myLastObject方法

```objective-c
#import "NSArray+Swizzle.h"

@implementation NSArray (Swizzle)

- (id)myLastObject{
    id ret = [self myLastObject];
    NSLog(@"**********  myLastObject *********** ");
    return ret;
}
```
2. 交换实现方法，并调用原方法

```objective-c
#import "NSArray+Swizzle.h"
#import <objc/runtime.h>
#import "NSArray+Swizzle.h"
int main(int argc, char *argv[]){
	@autoreleasepool {
	Method ori_Method = class_getInstanceMethod([NSArray class],@selector(lastObject));
	Method my_Method = class_getInstanceMethod([NSArray class], @selector(myLastObject));
	method_exchangeImplementations(ori_Method, my_Method);
	NSArray *array = @[@"0",@"1",@"2",@"3"];
	NSString *string = [array lastObject];
	NSLog(@"TEST RESULT : %@",string);
	return 0;
	}
}
```
3. 结果
```objective-c
2018-01-15 16:26:12.585 Hook[1740:c07] **********  myLastObject ***********
2018-01-15 16:26:12.589 Hook[1740:c07] TEST RESULT : 3
```

## 二、hook常用工具
- **class-dump**

dump目标对象的class信息的工具，利用Objective-C语言的runtime特性，将存储在Mach-O文件中的头文件信息提取出来，并生成对应的.h文件。

- **frida-trace**

动态追踪指定某些方法的调用堆栈。

- **Hopper disassembler**

适用于Mac操作系统的32位和64位的二进制反汇编器，反编译和调试，拥有伪代码以及控制流图(Control Flow Graph)的功能。

- **Interface Inspector**

Mac操作系统界面分析工具，用来分析界面上负责处理该状态的类，即Controller。

## 三、Hook实施
### 反射
通过反射可以根据类名获取类对象，进而调用应用程序类的实例方法、类方法
```objective-c
//类名获取类对象
Class _Nullable NSClassFromString(NSString *aClassName);
```
```objective-c
//调用类方法
- (id)performSelector:(SEL)aSelector;
- (id)performSelector:(SEL)aSelector withObject:(id)object;
- (id)performSelector:(SEL)aSelector withObject:(id)object1 withObject:(id)object2;
```
通过反射调用方法时，参数只能是任意类型id，而不能是基本类型或者是struct。
### 实例方法hook
代码封装示例：
```objective-c
+ (void)exchangeClass:(NSString *)className ori_Method:(SEL)ori_Selector new_Method:(SEL)new_Selector{

    Class QQ_Inject = NSClassFromString(@"QQ_Inject");

    Class ori_Class = NSClassFromString(className);

    class_addMethod(ori_Class, new_Selector, class_getMethodImplementation(QQ_Inject, new_Selector), "");

    Method ori_Method =  class_getInstanceMethod(ori_Class, ori_Selector);

    Method my_Method = class_getInstanceMethod(ori_Class, new_Selector);

    method_exchangeImplementations(ori_Method, my_Method);

}
```
### 类方法hook
类方法实现存储在元类中MetaClass，hook代码与实例方法有所不同，代码封装示例：
```objective-c
+ (void)exchangeMetaClass:(NSString *)className ori_Method:(SEL)ori_Selector new_Method:(SEL)new_Selector{
    Class QQ_Inject = objc_getMetaClass([@"QQ_Inject" cStringUsingEncoding:NSASCIIStringEncoding]);
    Class ori_Class = objc_getMetaClass([className cStringUsingEncoding:NSASCIIStringEncoding]);
    class_addMethod(ori_Class, new_Selector, class_getMethodImplementation(QQ_Inject, new_Selector), "");
    Method ori_Method =  class_getClassMethod(ori_Class, ori_Selector);
    Method my_Method = class_getClassMethod(ori_Class, new_Selector);
    method_exchangeImplementations(ori_Method, my_Method);
}
```
### hook时机
关于[hook时机](https://www.jianshu.com/p/036f502a2b15)有三种方式可选，分别为+load、+initialize和\_\_attribute\_\_((constructor))。attribute((constructor))是最适合的一般初始化选择。

```objective-c
__attribute__((constructor)) void addImagePackEntry (){

	[QQ_Inject exchangeMetaClass:@"MQAIOChatViewController" ori_Method:@selector(addImagePack:path:) new_Method:@selector(newAddImagePack:path:)];

}
```