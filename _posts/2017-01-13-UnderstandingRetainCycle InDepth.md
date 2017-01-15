---
layout:		post
title:			"温故而知新之循环引用的那些事儿"
subtitle:		"——常见的几种循环引用现象与解决方法"
date:			2017-1-13 23:00:00
author:		"Parsifal"
header-img:	"img/postresources/iOSDeveloper.jpg"
catalog:     true
abstract:    "- Block中的循环引用现象   
				- NSTimer中的循环引用现象    
				- NSNotificationCenter中的循环引用现象    
				"
tags:
- 侃侃技术 
- iOS
- Objective-C 
- Swift 
- 温故而知新
---
## 目录    
{:.no_toc}    
1.    
{:toc}    

## 扯扯闲话    
最近计划把以前写的一些技术类博文陆续搬过来这里，多少有点**温故而知新**的意思😄。当然，以前写的东西，现在看起来或多或少会有些觉得理解不够或者有误的地方，这回也会重新做下修正。同时，尽量补充些自己现在关于Swift方面的体会。这是两年前写的一篇博客，是关于OC中**循环引用（Retain Cycle）**的。先扔上来，督促自己快点修改。    

## TODOList   

<ul>
<li><input type="checkbox" disabled checked> 重新优化排版</li>
<li><input type="checkbox" disabled> 增删改</li>
<li><input type="checkbox" disabled> 补充Swift相关</li>
</ul>    


## Block中的循环引用现象    
#### Block的相关概念
{:.no_toc}    
我理解的block是加强版的函数指针（事实上，http://blog.devtang.com/blog/2013/07/28/a-look-inside-blocks/  文章中有分析到，block的数据结构中包含了一个函数指针来实现回调），很像匿名函数。通过block可以很方便的实现回调——block的一个优势就在于可以直接在调用处很直观地编写处理代码。OC里面，可以近似的把block看成一个对象，例如空Block应传值为nil，也有自己的isa指针，支持retain/release的内存管理方式等。

在Objective-C语言中，一共有3种类型的block：    

- **_NSConcreteGlobalBlock**——全局的静态block，不会访问任何外部变量。
- **_NSConcreteStackBlock**——保存在栈中的block，当函数返回时会被销毁。
- **_NSConcreteMallocBlock**——保存在堆中的block，当引用计数为0时会被销毁。    

其中ARC下，所有的StackBlock都会被copy到Heap，也就是只有两种Block了——_NSConcreteGlobalBlock和_NSConcreteMallocBlock。而全局区block与堆区block最明显的区别就是在于是否引用了外部变量，一旦引用了外部变量，block就将会被拷贝到栈上，堆区的block实际上就是对栈区的再次拷贝。
#### Block的声明与定义    
{:.no_toc}    
当时学习Block的时候，我是将block声明与C语言的函数声明、函数指针声明放到一起看，可以发现，与函数指针很像。    

```Objective-C
int max(int,int);//普通函数声明
int (*pmax)(int,int);//函数指针声明
int (^max)(int,int);//Block声明
```

下图是[官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1)中对Block的声明与定义的例子    
<img src="http://ojg3xdx9d.bkt.clouddn.com//blocks.jpg" width="556" height="228" align="center">


可以总结为：  

```Objective-C    
声明——返回值类型 (^Block名)(参数类型)；    
实现——^(参数类型 参数名) {return 返回值};    
```

通常会将Block进行**typedef**，例如图中例子就可以：    

```Objective-C
typedef int (^myBlock)(int);
```    

#### Block的使用   
{:.no_toc}    
Block的使用很简单，可以像C函数那样直接传参调用。例如上面例子，使用Block只要直接：    

```Objective-C
int result = myBlock(3);
```

这里要强调的一点便是Block中访问外部变量的情况，这也是Block的一个特色所在。还是拿上面的例子来说：  

```Objective-C
	int multiplier = 7;
	int (^myBlock)(int) = ^(int num) { return num * multiplier;};
```

此时，multipler是在myBlock外部定义的，也就是说实际上是与myBlock处于同一个作用域里的，并不是myBlock的内部参数。但myBlock却能够对其进行访问。这就涉及到前面说的Block的三种类型了。若Block内部有访问来自于外部的变量，则这个变量会被拷贝到栈上。也就是说Block内部访问到的变量实际上只是外部变量的一份值拷贝（长的相同，却不是一个人，参考C语言的形参与实参）。
但如果一定要对外部变量进行修改，官方也提供了一种方法，也就是用**__block**修饰变量（原理如同将变量的地址当作形参传入），这样就能修改外部变量值了。

#### Block的Retain Cycle现象    
{:.no_toc}    

在使用Block的时候，常常会造成**Retain Cycle**，而使得对象无法被释放。这里需要注意：Block的内部引用了其持有者，则只能引用其持有者的弱引用（为了防止弱引用的对象被提前释放，可在block内部进行强引用一次，作用域仅限于block内部，执行完立即被release）。
详见以下例子([点我看项目源码](https://github.com/parsifalc/BlockRetainCycleDemo))：    

```Objective-C
/******retain cycle test******/
    //1、这种情况下，虽然在block持有了self，但是不会造成retain cycle
    MyBlock myBlock = ^(NSString *str)
    {
        NSLog(@"局部的block 被执行str:%@  obj:%@", str, self);
        return str;
    };
   
    myBlock(@"1");
   
    //2、这种情况下，obj的block里面引用了obj，因而需要用弱引用。
    MyObject *obj = [[MyObject alloc] init];
    __weak MyObject *weakObj = obj;
   
    obj.myBlock = ^(NSString *str) {
        NSLog(@"obj的block被执行 str:%@ obj:%@", str, weakObj);
        return str;
    };
   
    /*
    obj.myBlock = ^(NSString *str) {
        NSLog(@"obj的block被执行 str:%@ obj:%@", obj, str);//错误的写法，会造成obj的retain cycle。这种错误造成的对象无法释放就比较难查到了，因为这个obj是局部的，并不会对当前controller的释放造成影响。这样的内存泄露应该尽量避免(可通过instrument的Leaks查找)。
        return str;
    };
    */
    obj.myBlock(@"2");
   
    //3、这种情况下，block的持有者--self，在block内部被引用了，同1，要用weakSelf
    __weak typeof(self) weakSelf = self;
    self.myBlock = ^(NSString *str)
    {
        __strong typeof(weakSelf) strongSelf = weakSelf;//在block内部对weakSelf进行强引用一次，避免weakSelf被提前释放，这次引用会在block结束后被释放；
        NSLog(@"self的block被执行 str:%@  obj:%@", str, strongSelf);
        return str;
    };
   
    /*
     self.myBlock = ^(NSString *str)
     {
     NSLog(@"self的block被执行 str:%@  obj:%@", str, self);//错误的写法，会造成self的retain cycle；
     return str;
     };
     */
    self.myBlock(@"3");
   
    /***RAC下的retain cycle****/
/*
 *总结，在RAC下订阅signal的时候，block里面引用到self，必须全部用weakSelf，引用到signal的订阅者，也必须用weakSubscriber。
 */

    MyTextField *myTextField = [[MyTextField alloc] initWithFrame:CGRectMake(50, 50, 100, 100)];
    myTextField.placeholder = @"hhhhhhhh";
    [self.view addSubview:myTextField];
   
    //4、这种情况下，也形成了retain cycle，而且RAC下面，在block里面引用self的话，也是会造成self的retain cycle，需使用weakObj
    __weak typeof(myTextField) weakTextField = myTextField;
    [myTextField.rac_textSignal subscribeNext:^(id x) {
        NSLog(@"obj:%@ self:%@", weakTextField, weakSelf);
    }];
   
    /*
    [myTextField.rac_textSignal subscribeNext:^(id x) {
        NSLog(@"obj:%@ self:%@", myTextField, weakSelf);//错误的写法，block的持有者myTextField形成retain cycle
    }];
    */
    /*
    [myTextField.rac_textSignal subscribeNext:^(id x) {
        NSLog(@"obj:%@ self:%@", weakTextField, self);//错误的写法，self形成retain cycle，与普通的不太一样，可能与RAC内部的实现机制有关
    }];
    */
   
    //5、这种情况下，同3，形成retain cycle，要用weakSelf
    @weakify(self);
    [self.textField.rac_textSignal subscribeNext:^(id x) {
        @strongify(self);
        NSLog(@"%@", self);
    }];
   
    /*
    [self.textField.rac_textSignal subscribeNext:^(id x) {
        NSLog(@"%@", self);//错误的写法，会造成self的retain cycle；
    }];
     */
    [self.view addSubview:self.textField];
```     

## NSTimer中的循环引用现象      

> Once scheduled on a run loop, the timer fires at the specified interval until it is invalidated. A non-repeating timer invalidates itself immediately after it fires. However, for a repeating timer, you must invalidate the timer object yourself by calling its invalidate method. Calling this method requests the removal of the timer from the current run loop; as a result, you should always call the invalidate method from the same thread on which the timer was installed. Invalidating the timer immediately disables it so that it no longer affects the run loop. The run loop then removes the timer (and the strong reference it had to the timer), either just before the invalidate method returns or at some later point. Once invalidated, timer objects cannot be reused.    

苹果官方文档中对NSTimer使用的建议，总而言之一句话：对于Repeat形式加入Runloop的timer，必须手动invalid，才能释放timer，从而释放timer中的target。如以下例子（[点我看源码](https://github.com/parsifalC/TimerRetainCycleDemo)）      

```Objective-C
    //not repeat---it will invalid by itself
    [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(log) userInfo:nil repeats:NO];

/*
    //这种写法是绝对不允许的 因为timer将无法invalid 你得不到这个timer
    [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(log) userInfo:nil repeats:YES];
*/
   
    //因为是repeat的timer，所以你必须手动invalid，而且这个操作必须在ViewController dealloc 方法执行前进行 例如 可以将timer的创建放在viewDidAppear里面，释放放在viewWillDisappear..
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(log) userInfo:nil repeats:YES];
```

这与KVO中的observer不一样，observer是不会被retain的。官方的描述：    

> Neither the receiver, nor an Observer, are retained. An object that calls this method must also call either the removeObserver:forKeyPath: or removeObserver:forKeyPath:context: method when participating in KVO.

## NSNotificationCenter中的循环引用现象    

```Objective-C 
[[NSNotificationCenter defaultCenter] addObserverForName:@"test" object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification *note) {
NSLog(@"%@", self);
}];
```    

这句话Block内部进行了对self的强引用，self被retain。而这个Block被释放的时机是**NSNotificationCenter**移除观察者。简而言之此处self要被释放的前提是移除观察者。那现在问题来了，要怎么移除观察者呢（我们并没有接收观察者对象）？参考苹果官方给的**NSNotificationCenter**的用法，更改代码如下：    

```Objective-C
//注册观察者
self.localCenter = 
[[NSNotificationCenter defaultCenter] addObserverForName:@"test" object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification *note) {
NSLog(@"%@", self);
}];

//移除观察者
- (void)dealloc
{
     [[NSNotificationCenter defaultCenter] removeObserver:self.localCenter];
}    
```    

这么写看上去很合理了。实际上我们又陷入了另一个陷阱。
**NSNotificationCenter**持有localCenter，localCenter持有Block，Block持有了self，self又持有了localCenter。很明显的retain cycle！可以看出切断这个cycle的关键就在于两点：
（1）何时将self移除观察者队列；在dealloc中移除显然是不行的，因为self的dealloc方法在Block释放self之前是肯定不会被调用的。若要移除必须在dealloc之前，至于在何处移除依需求而定了。
（2）在Block内部使用weakSelf，取消对self的强引用，这样self的dealloc方法就会被调用，在其中移除观察者，这么做是比较方便的。

综合比较，第二种方法是我们常见的也是比较好的做法。
正确的代码：    

```Objective-C
__weak typeof (self) weakSelf = self;
self.localCenter = [[NSNotificationCenter defaultCenter] addObserverForName:@"test" object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification *note) {
__strong typeof (self) strongSelf = weakSelf;
     NSLog(@"self:%@ note:%@", strongSelf, note);
}];

- (void)dealloc
{
     [[NSNotificationCenter defaultCenter] removeObserver:self.localCenter];
     NSLog(@"%s", __FUNCTION__);
}
```    

## 参考文章：    
{:.no_toc}    

<http://blog.csdn.net/crayondeng/article/details/20446891>    
<http://stackoverflow.com/questions/12699118/view-controller-dealloc-not-called-when-using-nsnotificationcenter-code-block-me>
