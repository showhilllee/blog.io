---
layout: post
title: swift与OC混编（创建混编工程）
date: 2016-01-25
categories: blog
tags: [写作,博客,千字计划]
description: 坚持每日千字。

---

> 原创文章转载请注明出处。

# 背景
***

Swift语言自推出以来苹果官方就开始努力开始推这门新语言。就在15年WWDC上更是把Swift进行了开源，瞬间star就开始不断飙升，截止今天（16年1月25）已经达到26059个star【[源码请点击这里](https://github.com/apple/swift)】，而且每天还都在不断增长，而且每天都会有Pull Requests。这足以说明全球开发者对这门语言的认可度。虽然现在在有些地方还不够完善，但是相信会在未来的某一天，Swift语言会成为一门主流开发语言。

# 混合开发
***

扯淡结束，进入正题。之前大家都熟悉了很久的OC语言，也许也开始尝试写一些Swift，但是这两个语言怎么进行混编呢？下面就说一下。Swift和OC的混编没有做的C与OC那样高度融合。

## 新建OC工程
***
新建工程不再多说，相信大家都很熟悉了，这里我们先拿OC项目中引用Swift来进行说明。

新建工程名为MixDemo，新建完工程以后可以不用进行任何修改和配置。我们先新建一个Swift类，新建方式与之前一样，只是在新建的时候注意选择语言为Swift。

![image](http://img.blog.csdn.net/20160126001315529)

新建Swift类名为SwiftDemo。点Create以后会出现这样一个弹出提示：

![image](http://img.blog.csdn.net/20160126001407002)

 **这是因为Swift和OC进行混编的时候需要一个中间的桥接文件。桥接文件中需要导入Swift类中引用到的OC类的头文件。**
 
 点击Create Bridging Header,工程中会多出两个文件，一个是SwiftDemo.swift文件和桥接文件MixDemo-Bridging-Header.h。新建完以后工程结构如图：
 
 ![image](http://img.blog.csdn.net/20160126001445623)## 简单的方法调用
***### OC调用Swift
***新建完成Swift类以后，我们来Build一下工程，perfect模拟器跑的很欢快，虽然什么代码也没有写。
下面我们在SwiftDemo.swift文件里写点东西。代码如下：
![image](http://img.blog.csdn.net/20160126001513034)
	@objc(SwiftDemo)

	public class SwiftDemo : NSObject {
    
	    public func logMe() {
	        print("log me");
	    }
    
	}
	
这里注意一下:

* 在Swift类中最好加`@objc`进行说明。因为如果不是继成自NSObject或NSObject的派生类，编译后将不会生成对应的转换类。在OC中也就找不到相应的声明，从而导致无法调用。

* 类和方法要声明成public才能在OC中调用到。如果你已经看过Swift，相信你对这个应该不陌生。

* 当你在Swift里添加一个public的类或者方法以后需要重新编译才会在OC端调用到。因为Swift需要重新编译转化为OC的“中间文件”，在“中间文件”中有了该类或者改方法OC才可以调用。关于这个“中间文件”下面会进行说明。

现在我们回到ViewController.m文件里。在OC文件中如果想要调用Swift类的方法的话需要导入一个头文件。该头文件在工程中不可见，默认格式为xxx-Swift.h，其中xxx为项目名。对于我们这个项目来说，也就是MixDemo-Swift.h

import该头文件以后我们就可以调用Swift的方法了。比如调用刚才我们写的SwiftDemo里面的logMe方法。上面提到修改Swift文件如果是public的话需要重新编译一下，好，那我们编译一下。perfect，还是没问题，继续。

这时候你就可以写你很熟悉的OC代码了。我们在viewDidLoad里添加代码如下：

	SwiftDemo* demo = [[SwiftDemo alloc] init];
    [demo logMe];
    
这时候你就会发现，当你写这段代码的时候，Xcode已经开始有提示了。是不是很神奇？

下面我们来看一下这个神秘的MixDemo-Swift.h文件。按住command键然后点击鼠标左键，进入该文件。直接滑动到该文件的最下面，你就会发现一件很有意思的事情：Xcode的提示并不是直接读取的Swift，而是由于这个“中间文件”。这个中间文件里有我们刚写的SwiftDemo和logMe函数。

![image](http://img.blog.csdn.net/20160126001552733)

> 思考:如果把objc括号里改成其他名称是否可以？修改以后再回到这个文件观察有什么变化。
然后Build一下，你会在控制台上看到打印了"log me"，这说明OC调用了我们刚才写的Swift的LogMe函数。### Swift调用OC
***Swift调用OC的时候需要在MixDemo-Bridging-Header.h文件中导入Swift会引用到的CO类的头文件。
首先我们在ViewController.h头文件中声明一个函数：
	- (void) logYou;
然后在实现文件中简单些一下实现：
	- (void)logYou {
    	NSLog(@"log you");
	}
 然后我们在MixDemo-Bridging-Header.h文件中#import "ViewController.h"文件,command+s保存一下当前的编辑。然后回到SwiftDemo.swift文件中，在LogMe函数里我们调用一下OC的logYou。
所以我们现在的逻辑是在OC的viewDidLoad中实例化SwiftDemo然后调用logMe,然后在SwiftDemo里的logMe里面实例化ViewController然后调用它的logYou。logMe代码如下：
	public func logMe() {
        print("log me");
        
        let vc = ViewController()
        vc.logYou()
    }
    
 然后再跑一下工程。两句话都打印了，很好。
 
 但是，这个MixDemo-Bridging-Header.h文件到底是个什么呢？
 
### 工程相关配置
***点击工程target然后点击Build Settings,然后在搜索框里输入swift,就会出现和Swift相关的配置信息，如图：
![image](http://img.blog.csdn.net/20160126001619405)你会在Object-C Bridging Header项后面发现一个熟悉的文件MixDemo-Bridging-Header.h。这个就是配置这个“中间文件”的地方。所以，现在你知道了这个文件配置的位置，以后你就可以自己任意修改成你喜欢的文件名字了，记得修改时文件路径问题，还有记得工程中要有与之对应的文件。
此外，一般情况下还需要注意修改的配置信息为：
* Embedded Content Contains Swift Code 项修改为YES。[点击这里查看stackoverflow上的先关问题说明](http://stackoverflow.com/questions/24163870/what-does-the-embedded-content-contains-swift-code-build-setting-in-xcode-6-do)
* Defines Module为YES
* Product Module Name为项目名称
# 结语
***

本文Demo可以在【[这里](https://github.com/showhilllee/MixDemo)】下载到
创建OC和Swift混编工程就说到这里。后面会说一下Swift怎么调用OC的类方法（静态方法）以及变参方法。
—End—
