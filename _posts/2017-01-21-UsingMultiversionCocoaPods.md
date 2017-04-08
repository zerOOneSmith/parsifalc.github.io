---
layout:		post
title:			"Cocoapods的多版本管理"
subtitle:		"——按项目使用Gemfile管理Cocoapods版本"
date:			2017-1-21 23:03:50
author:		"Parsifal"
header-img:	"img/postresources/codingwitdog.jpg"
catalog:     true
abstract:    "但不得不吐槽的是，CocoaPods的版本兼容一直不那么尽如人意。把老项目中的CocoaPods升级是非常痛苦的一件事，因为除了升级你本地的CocoaPods，还有一系列的工作（坑）等着你。Podfile或许需要按照最新规范更新，Podspec文件或许也需要跟着做调整，更或许有其他一些乱七八糟的error出现。"
tags:
- 侃侃技术
- iOS
- CocoaPods
---
## 目录    
{:.no_toc}    
1.    
{:toc}

## 扯扯闲话
首先，我们必须高兴，在全世界猿友们的共同努力下，[CocoaPods](https://cocoapods.org/)一直在不断更新着。这个帮我们解决很多依赖管理问题的开源库还在继续成长着。
<img src="http://ojg3xdx9d.bkt.clouddn.com//1487510496.png" width="1996" height="1224" align="center">
但不得不吐槽的是，CocoaPods的版本兼容一直不那么尽如人意。把老项目中的CocoaPods升级是非常痛苦的一件事，因为除了升级你本地的CocoaPods，还有一系列的工作（坑）等着你。Podfile或许需要按照最新规范更新，Podspec文件或许也需要跟着做调整，更或许有其他一些乱七八糟的error出现。在1.0.0以前，发布以前，0.35.0因其较稳定少出现莫名其妙的error而最受大家青睐。因而很多老项目至今也都是使用着0.35.0这个版本。所以就有了这样一个需求：老项目里继续用老版本的CocoaPods，新项目使用新版本的CocoaPods。索性我们的Cocoapods也已经考虑到了这点——使用Gemfile可以完美解决。
## 使用Gemfile管理多版本CocoaPods
先附上CocoaPods官网提供的方法链接：[Using a Gemfile](https://guides.cocoapods.org/using/a-gemfile.html)。以下做下使用方法的简单转述，如对原理感兴趣可到以上链接查看。


- 首先切换到想要更新Pod的工程目录，建立Gemfile文件，使用`bundle init`或者直接创建均可

```ruby
bundle init
```

- 编辑刚生成的Gemfile，这里我是用Sublime Text编辑的，subl是我定义的Alias，直接使用`vi Gemfile`也是一样的。编辑Gemfile文件，指定CocoaPods版本为'0.35.0'——`gem "cocoapods", '0.35.0'`

```ruby
subl Gemfile
```
<img src="http://ojg3xdx9d.bkt.clouddn.com//1487513656.png" width="1802" height="1146" align="center">

- 安装需要的CocoaPods版本，这一步bundle会读取Gemfile中的Gem进行安装


```ruby
bundle install
```

- 最后就可以使用指定版本CocoaPods安装了。这里若直接使用`pod install`，则默认是原全局默认的CocoaPods版本安装；若使用`bundle exec pod install`则会使用Gemfile内的版本安装。

```ruby
bundle exec pod install
```

## 其他方案管理多版本CocoaPods
以上便是使用Gemfile管理多版本CocoaPods的方法，很简单方便，只需要Clone工程后做一次设置即可。另外还有个方案是用RVM管理不同的Ruby版本对应的CocoaPods版本。这种方案的弊端是必须频繁地来回切换版本，不过也不失为一种方案。当然，最低效的方法是uninstall-install这种重复的操作。这里不再赘述。