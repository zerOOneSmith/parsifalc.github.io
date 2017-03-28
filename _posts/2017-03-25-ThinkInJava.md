---
layout:		post
title:			"Think In Java Notes"
subtitle:		"记录Java学习的一些小心得与体会"
date:			2017-3-25 19:10:00
author:		"Parsifal"
header-img:	"img/postresources/steve.jpg"
catalog:     true
abstract:    "学习之路永不止~从今天开始学习Java，当然主要是为了写Android。其实在Kotlin和Java之间犹豫了挺久，但现在还是以Java为主吧，毕竟涉及面更广一些。这篇博文主要记录自己学习期间的一些心得体会。所以有可能不是很系统，更多的只是一些简短的Tip。 
				"
tags:
- 侃侃技术
- Java
---
## 目录    
{:.no_toc}    
1.    
{:toc}

## 扯扯闲话
学习之路永不止~从今天开始学习Java，当然主要是为了写Android。其实在Kotlin和Java之间犹豫了挺久，但现在还是以Java为主吧，毕竟涉及面更广一些。这篇博文主要记录自己学习期间的一些心得体会。所以有可能不是很系统，更多的只是一些简短的Tip。学习主要是通过这两本很经典的资料--[Think In Java](https://www.gitbook.com/book/quanke/think-in-java/details)和[第一行代码](https://github.com/robertzhai/ebooks/blob/master/android/%E7%AC%AC%E4%B8%80%E8%A1%8C%E4%BB%A3%E7%A0%81%E2%80%94%E2%80%94Android.pdf)。其中第一行代码我是买的最新版本，上面链接是旧版本的。如果想买新版本的[可以看这里](http://blog.csdn.net/guolin_blog/article/details/52032038)。


## Think In Java
#### 基本语法
1）关于作用域的区别，在C/C++/OC中，小作用域与大作用域间变量允许同名，但Java为了避免混淆，直接将这种写法禁掉了。如下代码：

```java
{
	int x = 12;
	{
		int x = 11;//非法的，编译器报错
	}
}
```
2）Java里的**成员变量缺省值**里是一样的，都是空/0值，但局部变量的缺省值则是随机的。这点和OC是一样的。对于局部变量，Java编译器处理是必须赋值一个初始值，若没有则报error，这点和Swift目前的处理是一样的。

3）Java的方法只能作为类的一部分创建。Java是纯粹的完全面向对象的语言。因而在Java的世界中，你能写代码的地方实际都是在类中。

4）Java的构造器必须与类名完全一样，可带自变量参数。

```java
public class Child extends Object {
    Child() {
    }
}
```

#### 关键概念
1）内存回收方式
Java的内存回收是通过垃圾收集器（Garbage Collection）进行的。GC的流程大概是这样的：先找出在堆中还活着的对象，然后释放死对象所占用的资源，最后定期调整活对象的位置。常用的GC算法有：

- Mark-Sweep 标记清除
- Mark-Sweep-Compact 标记-整理
- Copying Collector 复制算法
- Mark-标记从”GC roots”开始扫描(这里的roots包括线程栈、静态常量等)，给能够沿着roots到达的对象标记为”live”,最终所有能够到达的对象都被标记为”live”,而无法到达的对象则为”dead”。效率和存活对象的数量是线性相关的。
- Sweep-清除扫描堆，定位到所有”dead”对象，并清理掉。效率和堆的大小是线性相关的。
- Compact-压缩对于对象的清除，会产生一些内存碎片，这时候就需要对这些内存进行压缩、整理。包括：relocate(将存货的对象移动到一起，从而释放出连续的可用内存)、remap(收集所有的对象引用指向新的对象地址)。效率和存活对象的数量是线性相关的。
- Copy-复制将内存分为”from”和”to”两个区域，垃圾回收时，将from区域的存活对象整体复制到to区域中。效率和存活对象的数量是线性相关的。

## 第一行代码




## 参考资料
- [Think In Java](https://www.gitbook.com/book/quanke/think-in-java/details)
- [第一行代码](https://github.com/robertzhai/ebooks/blob/master/android/%E7%AC%AC%E4%B8%80%E8%A1%8C%E4%BB%A3%E7%A0%81%E2%80%94%E2%80%94Android.pdf)
- [ANDROID学习之路](http://stormzhang.com/android/2014/07/07/learn-android-from-rookie/)
- [iOS 开发者的 Android 第一课](https://www.objccn.io/issue-11-1/)
- [Android Intents](https://objccn.io/issue-11-2/)
- [响应式 Android 应用](https://objccn.io/issue-11-3/)
- [Android 通知中心](https://objccn.io/issue-11-4/)
- [Android 中的 SQLite 数据库支持](https://objccn.io/issue-11-5/)
- [依赖注入和注解，为什么 Java 比你想象的要好](https://objccn.io/issue-11-6/)
- [谈谈Java内存管理](http://www.rowkey.me/blog/2016/05/07/javamm/)
