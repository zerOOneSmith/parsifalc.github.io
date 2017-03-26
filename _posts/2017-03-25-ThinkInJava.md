---
layout:		post
title:			"Think In Java Notes"
subtitle:		"记录Java学习的一些小心得与体会"
date:			2017-3-25 19:10:00
author:		"Parsifal"
header-img:	"img/postresources/core-java.jpg"
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

```Java
{
	int x = 12;
	{
		int x = 11;//非法的，编译器报错
	}
}
```



## 第一行代码




## 参考资料
- [Think In Java](https://www.gitbook.com/book/quanke/think-in-java/details)
- [第一行代码](https://github.com/robertzhai/ebooks/blob/master/android/%E7%AC%AC%E4%B8%80%E8%A1%8C%E4%BB%A3%E7%A0%81%E2%80%94%E2%80%94Android.pdf)
- [ANDROID学习之路](http://stormzhang.com/android/2014/07/07/learn-android-from-rookie/)
- [https://www.objccn.io/issue-11-1/](https://www.objccn.io/issue-11-1/)
- [Android Intents](https://objccn.io/issue-11-2/)
- [响应式 Android 应用](https://objccn.io/issue-11-3/)
- [Android 通知中心](https://objccn.io/issue-11-4/)
- [Android 中的 SQLite 数据库支持](https://objccn.io/issue-11-5/)
- [依赖注入和注解，为什么 Java 比你想象的要好](https://objccn.io/issue-11-6/)
