---
layout:		post
title:			"iOS开发工程师面试题整理"
subtitle:		"整理一下现阶段面试可能遇到的题目"
date:			2017-3-17 23:00:00
author:		"Parsifal"
header-img:	"img/postresources/Interview.jpg"
catalog:     true
abstract:    "首先Objective-C的内存管理机制是使用的引用计数管理，对象被引用时，retaincount加1，解除引用时retaincount减1，当retaincount为0的时候，系统会在下次autorelease pool drain的时候释放这些内存。在MRC时代，需要手动的做retain/release操作，这样不仅麻烦，而且容易造成各种内存问题，比如野指针、循环引用等。但iOS5的时候，这是苹果针推出了改良版管理方式-ARC。  
				"
tags:
- 
---
## 目录    
{:.no_toc}    
1.    
{:toc}

## 侃侃闲话
总结面试相关的一些题目。
## 内存管理
#### 1、ARC与MRC
首先Objective-C的内存管理机制是使用的引用计数管理，对象被引用时，retaincount加1，解除引用时retaincount减1，当retaincount为0的时候，系统会在下次autorelease pool drain的时候释放这些内存。在MRC时代，需要手动的做retain/release操作，这样不仅麻烦，而且容易造成各种内存问题，比如野指针、循环引用等。但iOS5的时候，这是苹果针推出了改良版管理方式-ARC。相对MRC而言，最大的好处是由编译器负责做retain和release操作，减少了很多内存问题。引用计数的缺点是无法进行相互引用，但OC里有弱引用的概念来解决这个问题。与引用计数的管理方法相对应的是，Java的垃圾回收策略（GC）。常见的GC方法有标记-清除算法（Mark-Sweep），缺点是会造成很多空间碎片，其他方法还有复制算法等。

#### 2、请解释以下keywords的区别： assign vs weak, __block vs __weak
assign与weak：共同点是都不会增加retaincount，区别是a)assign不仅能用于基础数据类型，也能用于OC对象，但weak只能用于OC对象；b)weak是弱引用，在其所引用的对象被释放后，会将变量置为nil，不会有野指针的现象，通常用于Delegate来避免循环引用。c)weak只能用于ARC；
__block与__weak：__weak是弱引用，通常用于block外部避免循环引用的；__block在block外部使用的话，能提高变量的作用域。a)__block不管是ARC还是MRC模式下都可以使用，可以修饰对象，还可以修饰基本数据类型；b)__weak只能在ARC模式下使用，也只能修饰对象(NSString)，不能修饰基本数据类型(int)；c)__block对象可以在block中被重新赋值，__weak不可以；d)__block对象在ARC下可能会导致循环引用，非ARC下会避免循环引用，__weak只在ARC下使用，可以避免循环引用。e)ARC下__block会增加retain count，MRC下则不会；
__weak与weak：一个是变量所有权修饰符，一个是属性修饰符。

#### 3、描述一个你遇到过的retain cycle例子
block的循环引用、delegate的循环引用、NSTimer的循环引用

## 并发编程
#### 1、使用atomic一定是线程安全的吗？
atomic与nonatomic是属性的修饰符，atomic的线程安全是系统基于setter方法做@synchronized(self){//code }，但这种做法是需要消耗很多资源的，一般iOS开发我们使用nonatomic。各种锁的使用https://www.zybuluo.com/MicroCai/note/64272 常用锁：NSLock、@synchronized（隐式异常处理）、NSRecursiveLock、NSConditionLock、NSDistributedLock、pthread_mutex_t、GCD的同步锁（使用串行同步队列和dispatch_sync）以及信号量（生产者消费者模式）等。

#### 2、NSThread、GCD和NSOperationQueue的比较
`NSThread`是对`POSIX thread`的封装，也就是`pthread`，它可以直接接触线程，控制线程，如果不加控制，很容易开辟出很多线程，消耗很多资源；

`GCD`和`NSOperationQueue`都是基于线程池的，不直接接触线程，开发者只需要把task扔给线程池，让线程池去分配执行就好了。其中`NSOperationQueue`是对`GCD`的更高一层的封装，可以取消Operation的执行。但写起来相交`GCD`没有那么直观可靠。`GCD`使用起来很方便，通过Block回调，但需要注意retain cycle现象。

#### 3、GCD里面有哪几种Queue？你自己建立过串行queue吗？背后的线程模型是什么样的？
**GCD里面有哪几种Queue**

可以说分为两类，串行和并行，串行Queue里有一个特殊的队列，为**main dispatch queue**，为应用程序的主线程，是UI处理的唯一合法线程。
> Serial：又叫private dispatch queues，同时只执行一个任务。Serial queue常用于同步访问特定的资源或数据。当你创建多个Serial queue时，虽然各自是同步，但serial queue之间是并发执行。
> 
>Concurrent：又叫global dispatch queue，可以并发的执行多个任务，但执行完成顺序是随机的。
>
>Main dispatch queue：全局可用的serial queue，在应用程序主线程上执行任务。

**你自己建立过串行queue吗**

建立过，用于后台数据库的读写操作处理。

**背后的线程模型是什么样的？**

线程模型是苹果负责处理维护的线程池，所有的队列都只能从这个池子里拿线程，开发者不直接接触线程，而是把任务扔给线程池负责处理。

#### 4、为什么iOS里，UI必须在主线程更新？
1）首先，多线程编程里，资源竞争始终是一个问题，而在UIKit这种级别的框架里，确保线程安全会是一个重大的任务，如果要做的话，成本也是非常大的。因而苹果并没有将UIKit完全做成线程安全的。早期版本，几乎所有的绘图方法都是非线程安全的，但在iOS4中苹果将大部分绘图的方法和诸如`UIColor`和`UIFont`这样的类改写为了后台线程可用。有了这个前提后，为了保证UI的绘制不出现相关的竞争问题，苹果一直提倡将UI更新写到主线程里。


2）其次，如果在非主线程操作UI的话，其实不一定会发生崩溃，但是可能会延迟UI更新。因为苹果会将非主线程内的UI操作延迟到该子线程释放后，再从主线程里调用更新的函数栈。这时机是很不确定的。

## Runtime
**Runtime是OC面向对象的基础，同时支持了动态特性。**
#### 1、`+(void)load`和`+(void)initialize`有什么用处？
load方法是runtime在加载类后，像每个类和分类发送的消息，顺序是父类-子类-分类（如果都是分类的话，按照编译顺序调用），其发送时机是在main函数之前，在OC里，通常在这个地方做method swizzlling或者使用Router方式做解耦的时候，通常也会在这里做URL注册；initialize方法是一个惰性方法，被调用的前提是，该类有方法被调用，如果该类一直没有被使用过，则这个方法一直不会被调用。相同点是，两个方法都是线程安全的。


#### 2、为什么其他语言里叫函数调用， objective c里则是给对象发消息（或者谈下对runtime的理解）
c\c++的函数是静态调用的，在编译时就已经确定下来了；而Objective-C却是有动态特性的。OC的编译器clang/llvm对函数的编译是全部编译成msgSend函数，通过传递对象和selector及参数来在运行时寻找真正的IMP并执行。

oc消息发送的全过程：runtime通过消息接收对象的isa，找到类的结构体（会沿着继承链一直往上找），从结构体里找到方法列表，再从方法列表里找相应的selector，最后通过selector找到具体的IMP。
a)实例方法：从消息接收对象的Class对象方法列表里开始寻找，找不到则一级级往父类找，直到根类对象-》NSObject；
类方法：从消息接收对象的MetaClass方法列表里开始寻找，找不到则一级级往父类找，直到根MetaClass，其中MetaClass的isa指向自己，SuperClass指向NSObject。

如果找不到selector对应的话会走以下3个流程：
a)`-resolveInstanceMethod`或`+resolveClassMethod`，可在此处动态添加该selector的IMP；
b)快速转发forwardingTargetForSelector，可将该消息转发给其他对象，返回非空对象后，重启消息发送；
c)完整消息转发forwardInvocation，这里需要先提前实现方法签名`methodSignatureForSelector`，runtime会调用这个方法并生成一个NSInvocation对象，再在forwardInvocation传入之前生成的NSInvocation对象，在这个方法转发消息对象。

#### 3、什么是method swizzling?
method swizzling是利用oc的runtime机制，交换方法IMP的一种手段。比如经常被提到的面向切面编程里，就经常使用这个。再比如，网易前阵子出的Baymax自防护机制，就是通过各种method swizzling实现的。

## CocoaTouch
<img src="http://ojg3xdx9d.bkt.clouddn.com//1489905484.png" width="873" height="1159" align="center">

#### 1、UIView和CALayer是啥关系?
1）首先，UIView是继承自UIResponder的，是绝大部分（除UIMenuItem\UIBarItem等外）视图类的基类。可以处理各种交互事件，包含一个CALayer的属性来负责绘制功能。可以把UIView当做CALayer的一个管理器来理解，实际上它就是CALayerDelegate。由于UIView处理的事情更多，所以其带来的性能损耗也是要明显大于CALayer的，所以在做一些性能优化的时候，常常把明显不需要处理事件的视图替换成CALayer。


2）UIView的frame等坐标系是直接返回其CALayer属性的，而CALayer属性的坐标体系就比较复杂了，除了frame外，还有anchorPoint\transform等。


3）CALayer内部维护着三分layer tree,分别是 presentLayer Tree(动画树),modeLayer Tree(模型树), Render Tree (渲染树),在做 iOS动画的时候，我们修改动画的属性，在动画的其实是 Layer 的 presentLayer的属性值,而最终展示在界面上的其实是提供 View的modelLayer

#### 2、loadView是干嘛用的？
loadView是手写ViewController时，生成当前VC视图的惰性方法，如果要自定义View，可以直接复写这个方法。如果是IB创建的VC，则就不需要了。自定义的方法无需调用super方法。

#### 3、viewWillLayoutSubView你总是知道的。。
controller layout触发的时候，开发者有机会去重新layout自己的各个subview。说UI熟悉的一定要知道。主要应用场景在于横竖屏切换的时候。

## 优化
[优化这部分强推YYKit作者的这篇博文](http://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
#### 1、如何高性能的给UIImageView加个圆角？
UIImageView的圆角造成低效的主要原因是设置了cornerRadius后又设置了maskToBounds，造成了离屏渲染（对mask，drawRect等操作均会造成离屏渲染），要高效地完成圆角方案需要让CPU更多地参与进来以减少GPU的工作。比较好的方案是：通过CPU在非主线程绘制好圆角的图片，再返回主线程赋值给UIImageView。简单的方法是Rasterize，将离屏渲染移到CPU做，但会增加CPU负担。取巧的方案是让美工切一个中间圆形镂空的图，但这样需要背景与图片颜色一样。

- 离屏渲染：当使用离屏渲染时，GPU 第一次会混合所有图层到一个基于新的纹理的位图缓存上，然后使用这个纹理来绘制到屏幕上。现在，当这些图层一起移动的时候，GPU 便可以复用这个位图缓存，并且只需要做很少的工作。

- 关于卡顿：显示器显示画面，是类似一种电子枪扫描屏幕的机制，从上到下，从左到右，每行的开始会有HSync（横向同步信号），每次开始新的一轮会有VSync（垂直同步信号）。现在的GPU渲染一般都是有缓存机制的。CPU将显示内容计算好之后，传给GPU完成渲染并放入缓冲区，GPU将帧数据放到缓存区后，同时会将视频控制器的指针指向最新的缓存区，而视频控制器就是通过VSync从缓存区不断读取渲染好的帧画面。这样只要是GPU速度快于视频控制器的话，就会有页面分裂的现象，即上一帧显示器还没显示好，下一帧就开始显示了。为了解决这个问题，引入了垂直同步的机制。不开垂直同步机制的时候，GPU只管自己埋头干活，干好了就把视频控制器挪到新的缓存区，而不管视频控制器的进度。但开了垂直同步机制的时候，GPU会等视频控制器发出VSync信号后，再进行渲染了。**iOS目前就是开启垂直同步机制且使用双缓冲的。**垂直同步机制解决了分裂问题，但效率会降低一些，且会出现了掉帧的现象。**卡顿的原因在于CPU的内容计算和GPU的内容渲染工作无法在一个VSync时间内完成。因此，需要协调好CPU和GPU的工作分配。**

- **CPU消耗原因：**对象创建、对象调整、对象销毁（可用GCD放到后台线程销毁）、布局计算、autolayout、文本计算、文本渲染、图片解码、图像绘制；
- **GPU消耗原因：**纹理的渲染、视图的混合、图形的生成

**可使用Instrument-Core Animation检测CPU和GPU的使用情况。**

#### 2、使用drawRect有什么影响？（这个可深可浅，你至少得用过。。）
drawRect方法依赖Core Graphics框架来进行自定义的绘制，但这种方法主要的缺点就是它处理touch事件的方式：每次按钮被点击后，都会用**setNeddsDisplay进行强制重绘**；而且不止一次，每次单点事件触发两次执行。这样的话从性能的角度来说，对CPU和内存来说都是欠佳的。特别是如果在我们的界面上有多个这样的UIButton实例。drawRect会利用CPU生成offscreen bitmap，从而减轻GPU的绘制压力，用这种方式最UI可以将动画流畅性优化到极致，但缺点是绘制api复杂，offscreen cache增加内存开销。

#### 3、都做过哪些性能优化？
**1）App的IPA包瘦身**

- 资源的瘦身，包括去除1x的图片资源（不支持4以下的机型）
- 图片资源通过ImageOptim去除无意义的数据
- 利用LSUnusedResources扫描删除无用的资源文件
- BitCode的使用（BitCode是一种中间代码，如果将BitCode传到iTunes，苹果会根据不同的机型和系统版本，对我们的二进制文件再优化，下发不同的IPA包）
- 编译器优化级别（慎用）
- 尽量使用.xcassets管理（苹果在iOS9后，对xcassets管理的资源，如果设备只支持2x，就不会下载3x）。
- 可能还有IconFont等图片方案
- 去除无用的代码
- 去除占用空间太大的第三方库


**2）App速度的优化**

- [启动速度优化](https://techblog.toutiao.com/2017/01/17/iosspeed/)，分main函数之前（主要是一些链接和加载，去除无意义的framework，去除无用类和函数等）和main函数之后（首页最好使用代码加载，不用Xib、除了必要的如第三方平台初始化等操作，不必要操作都异步或者延迟进行、梳理网络请求）
- 内存优化，建议开发期使用`MLeaksFinder`做检测，阶段性使用instruments-leaks做完整检测，解决项目中的RetainCycle引起的Leaks；使用instruments-allocations检测abandoned memory。（内存分为三类，Leaked Memory（能够直接用Leaks工具检测出的内存泄漏，如retain cycle这类），Abandoned Memory（不能用Leaks看到的，只能手动一个个去看alloc和release的数目，Allocations里可以观察Persistent\Transient\Total这几个量），Cached Memory（适当的缓存可以提高效率，但需要控制缓存的大小））使用Instruments-Analyze可以协助定位代码的逻辑和内存问题。在循环便利中，如果临时变量比较多，手动创建AutoreleasePool，即时释放内存。`imageName:`和`imageWithContentsOfFile:`合理使用，`NSDateFormatter`的重用
- 卡顿优化，使用Instrument-Core Animation和OpenGL ES Analysis做深度优化，主要是减少主线程的任务，且合理分配CPU和GPU的工作，进行适当缓存以空间换时间。

#### 4、讲讲你用Instrument优化动画性能的经历吧（别问我什么是Instrument）
同如何高性能添加圆角，主要是GPU和CPU的工作分配。


## 第三方库
#### 1、AFNetworking或者SDWebImage里面给UIImageView加载图片的逻辑是什么样的？
这两个库都是同时支持内存缓存和本地缓存的，内存缓存都是基于NSCache，AFNetworking使用的NSURLCache做本地缓存，缓存的是RawData，而SDWebImage是自己处理的本地缓存逻辑，缓存的是Image数据。也就是说在读缓存的时候，SDWebImage会少了一步图片解析的过程，这也是更多人使用SDWebImage的原因。除了缓存的方案不同，其他异步下载等操作都是类似的，大致如下：

- 调用UIImageView的分类方法后，若先显示Placeholder图片，然后开始获取图片逻辑
- 如果图片已经在SDImageCache内存缓存中，则直接返回图片，若图片没有在内存缓存中，则添加一个Operation开启本地缓存的异步查询逻辑
- 根据URLKey生成的MD5作为文件名，找到则把图片存入内存缓存，并回调回主线程，若找不到，则新开一个Operation添加进下载的队列里，由NSURLSession开启下载数据
- 回调下载进度，下载完成后，开启一个Operation走解码流程
- 解码完成后，层层回调回主线程调用处，并且缓存到内存里，同时开启另一个operation进行磁盘缓存逻辑
- 内存缓存有设置最大缓存空间，磁盘缓存同样也有，以上的每次写缓存操作，都会先计算是否有剩余空间，空间不足的情况下，删除较早的那些数据。另外SDImageCache还提供了删除最近n天的缓存数据逻辑，默认是7天。

## 持久化
#### 1、用过coredata或者sqlite吗？读写是分线程的吗？遇到过死锁没？咋解决的？
SQLLite使用的是FMDB库，读写是在一个独立线程里，以前有遇到过死锁的情况，主要是dispatch_sync到main queue的时候，会造成死锁。另外，如果两个资源互相等待的情况，也会出现思死锁。

#### 2、Realm的优缺点和使用过程中遇到的问题
**优点**

- API简单，光速上手。甚至不用写sql，大家简单阅读下文档，就可以在实际项目中开用了。升级、迁移等都有非常成熟的接口。
- 性能优秀。简单看过原理，相比于传统数据库 链接 - 查询 - 命中 - 内存拷贝 - 对象序列化 的复杂过程，Realm采用基于内存映射的Zero-Copy技术，速度快一个数量级。而且内部采用了类似git的对象版本管理机制，并发的性能和安全性也不错。
- 响应式。Realm的查询结果是随数据库变化实时更新的（要求对象在Run Loop线程中），配合KVO或者Realm自带的ObjectNotification，可以轻松构建及时反映数据变化的响应式UI，这点和目前主流的响应式框架（ReactNative，ReactiveCocoa）应该是天作之和了。但是要求使用者改变思路，不然会出现很多诡异的bug。
- ORM。虽然Realm自己号称是『为移动开发者定制的全功能数据库』，但是其中确实包含ORM的很多特性，不用手动写中间层了，取出来就是新鲜活泼的对象，everyone is happy。


**缺点：**

- framework比较大，基本在7MB左右，相对的FMDB，才200KB
- 每个类都必须继承于Realm的基类，早期版本还没有提供JSON转化模型的功能，需要自己手写一套，不过新版本应该增加上了。
- 关于数组的使用，只能用RLMArray，侵入性太强了，后期如果换其他方案，代码改动量会很大
- RealmObject自带线程保护功能，只能在创建它的线程中访问，在子线程中不能访问。也就是说，如果你在主线程中new了一个RealmObject对象 user，那么在子线程中是访问不了user对象的。
要想在子线程中访问，必须先将user存入Ream中，然后在子线程中query出来。
- 如果Realm关闭，所有查询得到的RealmObject都不能使用了。如果想在子线程中去查询数据，然后在主线程中使用是无法做到的。所以Realm提供的异步查询就很重要了...
- 如果想在Realm.close()之后继续操作查询得到的对象，只能复制一份数据传出来。为防止Realm忘记关闭，个人喜欢将Realm的开启和关闭封装在一个函数中。但是realm Colse掉之后，query得到对象就不能访问了，所以只能复制一份数据传出来。这个比较坑，Realm开发者是为了它的一个特色功能Auto-Update，即自动更新查询到的数据，特意让查询得到的数据与数据库中的数据保持了同步，所以Realm一关，外面的数据也用不了。而且，这个Auto-update暂时还无法关闭，stackOverFlow上有说以后可能会提供关闭这个功能的方法。如果你的RealmObject非常复杂，要copy一份数据将会很麻烦...而且这还不是最坑的，最坑的是下面这条。
- 如果直接修改或删除query得到的数据，必须在transaction中完成...
也就是说，你根本不能把query返回的对象，当成普通对象去赋值或删除，如果想要直接操作...ok，把对象copy一份传出来...

## 网络协议
#### 1、http的post和get啥区别？（区别挺多的，麻烦多说点）

- get请求可以直接以浏览器访问，post则不能；
- post可以往body里面填更多的数据，get则不能；
- 浏览器一般会限制get的URL长度，无法传递较大数据，一般限制1024Byte？
- 服务端对这两个请求的处理也不一样；
- post安全性稍微高一点，主要是数据没有放在url上；

## 算法与数据结构
#### 1、我知道你大学毕业过后就没接触过算法数据结构了，但是请你一定告诉我什么是Binary search tree? search的时间复杂度是多少？我很想知道！
Binary search tree:二叉搜索树。 

- search：时间复杂度为O(h)，h为树的高度
- traversal：时间复杂度为O(n)，n为树的总结点数。
- insert：时间复杂度为O(h)，h为树的高度。
- delete：最坏情况下，时间复杂度为O(h)+指针的移动开销。

可以看到，二叉搜索树的dictionary operation的时间复杂度与树的高度h相关。所以需要尽可能的降低树的高度，由此引出平衡二叉树Balanced binary tree。它要求左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。这样就可以将搜索树的高度尽量减小。常用算法有红黑树、AVL、Treap、伸展树等。

## 常用工具的原理
#### 1、git：最受欢迎的版本管理工具。
**原理：**git是一个分布式的版本管理工具。每个提交都会生成一个快照。通过HEAD指针指向不同的快照来管理版本记录。每个提交的指针都是SHA-1生成的一个字符串。

#### 2、xUnique：避免多人开发中project文件冲突的神器。一个由Python编写的脚本。
[Github地址](https://github.com/truebit/xUnique)

**原理：**Xcode管理文件主要是通过project文件进行。对于每个文件，project会生成一串16进制数据，并通过它进行文件引用。但Xcode默认生成这个数据串的算法，对于同一个文件在不同机子上会生成不同的数据。而这也是导致冲突的根本原因。xUnique会将project文件转为JSON格式，给每个16进制数据分配一个路径且对所有文件路径进行MD5加密，从而生成心得UUID，替换原来Xcode生成的那些16进制数据UUID，最后对这些引用文件进行排序。由于是MD5，就保证了每台机子是一样的。

#### 3、JLRoutes：URL处理助手
**原理：**保存一个全局的Map，key是url，value是对应存放block的数组，url和block都会常驻在内存中，当打开一个URL时，JLRoutes就可以遍历 , 这个全局的map，通过url来执行对应的block。

#### 4、JSPatch：基于JavaScriptCore的动态化方案。
**原理：**基础是OC的runtime机制——所有方法的的调用/类的生成都是在运行时进行的，且我们可以通过类名/方法名反射得到相应的类/方法。基本原理大致为：JS传递字符串给OC-》OC通过runtime调用、替换或增加方法-》OC将结果返回给JS。
1、方法调用：（1）require：在JS全局作用域生成一个同名变量，变量指向一个包含clsName且注明是OC类的对象；（2）给JS基类添加一个元方法，将所有JS脚本里的方法调用全部转到调用这个元方法里，在元方法里根据对象类型进行分发；（3）通过JavaScriptCore进行OC和JS互相传递消息；（4）对于一个自定义id对象，JavaScriptCore 会把这个自定义对象的指针传给 JS，由于无法区分判断一个对象是否是OC指针，所以在OC对象返回给JS之前进行手动封装为一个@{@“_obj”:obj}的字典，通过这样的方式确定指针为OC对象；（5）做必要的类型转换
2、通过method swizzling

#### 5、MLeaksFinder：实时查找内存泄漏的小助手
[Github地址](https://wereadteam.github.io/2016/02/22/MLeaksFinder/)

**原理：**给NSObject添加了两个方法，在UIViewController被Pop/Dismiss回来的时候，调用willDealloc方法即可。


```swift
- (BOOL)willDealloc {
    __weak id weakSelf = self;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [weakSelf assertNotDealloc];
    });
    return YES;
}
- (void)assertNotDealloc {
     NSAssert(NO, @“”);
}
```

#### 6、ImageOptim：图片瘦身工具
**原理：**主要是删除图片中的对客户端无用的可删数据，如元数据、伽马值等等

#### 7、LSUnusedResources：检索应用中未使用资源的工具
**原理：**通过扫描特定的函数是否引入资源名来检索，但仍可能有误删嫌疑，需要手动校验


## 其他
#### 1、麻烦你设计个简单的图片内存缓存器（移除策略是一定要说的）
简单的可以直接用NSCache做的，如果要手动实现的话，我会封装一个图片的模型对象（key、totalBytes等数据），通过一个单例对象管理数组存储这些模型对象，可以设置删除策略为FIFO或LIFO，可设置缓存空间大小。


## 参考资源

- [MrPeak-iOS中级面试题](http://mrpeak.cn/ios/2016/01/07/push)
- [答MrPeak的iOS面试题](http://blog.csdn.net/hanangellove/article/details/45033453)
- [iOS保持界面流畅的技巧](http://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
- [iOS并发编程](https://github.com/ming1016/study/wiki/iOS%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B)
- [Realm踩过的那些坑](https://www.zhihu.com/question/30298585/answer/93339976)
- [招聘一个靠谱的iOS](https://github.com/ChenYilong/iOSInterviewQuestions)
- [线程安全类的设计](https://objccn.io/issue-2-4/)
