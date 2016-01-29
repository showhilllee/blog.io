---
layout: post
title: swift与OC混编（方法调用）
date: 2016-01-25
categories: blog
tags: [写作,博客,千字计划]
description: 坚持每日千字。

---

> 原创文章转载请注明出处。

# 背景
***

在[上一篇文章](http://showhilllee.github.io/blog/2016/01/25/swift-and-oc/)中简单讲述了怎么创建Swift和OC的混编工程。本篇讲一下Swift和OC的混编工程中的方法调用。

# OC调用Swift
***

OC调用Swift方法比较简单。但是需要注意以下几点：

* 1.需要在当前OC类里导入xxx-Swift.h头文件，其中xxx为项目名称（与你的项目配置相关，具体配置方式见上一篇文章）

* 2.OC类里仅可以调用public的Swift方法和变量

* 3.Swift类最好用@objc(xxx)进行描述


暂时想到这几点，如有疏漏请留言指出，不胜感激。

剩下的调用方式就和普通的OC之间调用方式类似了。

## OC调用Swift实例方法
***

继续用上一篇文章的例子进行说明，例如在ViewController.m类里调用Swift的logMe实例方法，就可以这么写：

	SwiftDemo* demo = [[SwiftDemo alloc] init];
    [demo logMe];

## OC调用Swift静态方法
***

首先先在SwiftDemo.swift文件中声明一个静态方法：

	public static func swiftStaticFunc(log: NSString) {
        print(log);
    }


然后回到ViewController.m类里调用该方法（记得编译一下才可以吆~）

桐乡的，调用方式和OC之间的调用类似：

	[SwiftDemo swiftStaticFunc:@"oc call swift static func"];


# Swift调用OC
***

前一篇文中也已经说过需要有一个桥接文件Swift才可以调用OC相关方法以及属性等，这里不再赘述，忘记了的可以再回去看一下。

## Swift调用OC实例方法
***其实前面文中已经举例说明了调用方法。这里再简单重复一下。
在SwiftDemo.swift类里调用ViewController.m类里的logYou方法，swift调用代码如下：
	let vc = ViewController()
    vc.logYou()声明一个变量vc，也就是ViewController的实例对象。然后用vc对象调用实例方法logYou。
## Swift调用OC多参方法
***

首先先在ViewController.h中声明一个OC的多参方法：

	- (void) logMe:(NSString*)logMe logYou:(NSString*)logYou;
	
在.m文件中进行一下实现：	- (void)logMe:(NSString *)logMe logYou:(NSString *)logYou {
	    NSLog(@"%@--%@", logMe, logYou);
	}在SwiftDemo.swift文件中调用方法如下：
	vc.logMe("log me", logYou: "log you")
方法从第一个参数开始都要写在括号里。## Swift调用OC静态方法
***

首先先在ViewController.h中声明一个OC的静态方法：

	+ (void) ocStaticFunc:(NSString*)log;

然后在.m文件中简单些一下实现：

	+ (void)ocStaticFunc:(NSString *)log {
	    NSLog(@"%@", log);
	}回到SwiftDemo.swift文件中，用swift调用OC的静态方法。
	ViewController.ocStaticFunc("swift call oc static func")

## Swift调用OC变参方法
***

在某些需求情景下，需要用到变参函数。简单举个例子：

	+ (void) stringParams:(NSString*)params,...;

这种例子在系统函数中也可以见到。比如常用的NSString的一个方法：

	- (instancetype)initWithFormat:(NSString *)format, ... NS_FORMAT_FUNCTION(1,2);
	
OC的调用方法就不再重复了。这里说一下Swift怎么调用OC的变参方法。

首先，**Swift不能直接调用OC的变参方法**。

如果必须要用到，则需要对函数进行简单修改。

拿上面刚说到的`stringParams:`方法举例。需要把方法的写法改为：
	+ (void) stringParams:(NSString*)params args:(va_list)args;
函数的具体实现：
	+ (void) stringParams:(NSString *)params args:(va_list)args {
    
	    va_list args_copy;
	    
	    __va_copy(args_copy,args);
	    NSMutableString* format = [NSMutableString stringWithString:@""];
	    while (va_arg(args, NSString*))
	    {
	        [format appendString:@"%@,"];
	    }
	    va_end(args);
	    
	    if(format.length>0)
	        [format deleteCharactersInRange:NSMakeRange(format.length-1,1)];
	    
	    NSString* newFormat = [NSString stringWithFormat:@"%@",format];
	    NSString * result = [[NSString alloc]initWithFormat:newFormat arguments:args_copy];
	    va_end(args_copy);
	    NSLog(@"%@", result);
	}在Swift中的调用方式：
	let args: [CVarArgType] = ["i'm", " showhilllee"]
        withVaList(args) {
            (pointer: CVaListPointer) in
            return ViewController.stringParams("%@,%@", args:pointer)
        }
      
 当然，还有其他方式来实现。可以尝试找一下方法。# 结语
***

本文Demo可以在【[这里](https://github.com/showhilllee/MixDemo)】下载到

—End—
