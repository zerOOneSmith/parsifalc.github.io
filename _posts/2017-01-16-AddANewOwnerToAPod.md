---
layout:		post
title:			"Not Allowed To Push New Versions For This Pod"
subtitle:		"——为Pod添加管理账号的方法"
date:			2017-1-15 23:00:00
author:		"Parsifal"
header-img:	"img/postresources/keyboard.jpg"
catalog:     false
abstract:    "晚上在更新自己的一个旧轮子Pod的时候，遇到个有趣的问题，记录分享一下。事情是这样子的：由于更换了邮箱，我现在写轮子都是用的新邮箱作为CocoaPods的发布账号。今天更新这个轮子是前两年写的，也就是意味着是Pod的所有者是我的旧账号。于是也就有了下面这个略尴尬的一幕。"
tags:
- 侃侃技术 
- iOS
- CocoaPods 
- 走过的那些坑
---

晚上在更新自己的一个旧轮子Pod的时候，遇到个有趣的问题，记录分享一下。事情是这样子的：由于更换了邮箱，我现在写轮子都是用的新邮箱作为CocoaPods的发布账号。今天更新这个轮子是前两年写的，也就是意味着是Pod的所有者是我的旧账号。于是也就有了下面这个略尴尬的一幕。     

<img src="http://ojg3xdx9d.bkt.clouddn.com//1484573796.png" width="2560" height="698" align="center">    
WTF!居然说我没有权限😂！不过还好，CocoaPods的给出的提示挺友好的。信息展示的很充分。很自然的可以联想到解决方法：    

- 方法1：继续使用原账号发布更新；
- 方法2：让新账号拥有这个Pod的更新权利；

毫无疑问，我需要选择第二个方法。因为不可能老是频繁切换着账号来做这些维护。这样有了方法后，需要去寻找操作步骤了。同样的，有问题，先看CMD的help。     
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484573961.png" width="2560" height="936" align="center">
WOW！CocoaPods提供的COMMANDS里直接就有这些操作方法了。这样连Google都省了。我只要先登录原来的账号，然后把这个Pod授权给新账号，再切回新账号就可以了。
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484576277.png" width="2558" height="494" align="center">
切换账号的需要邮件验证。
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484580203.png" width="1688" height="566" align="center">
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484573943.png" width="2560" height="774" align="center">

最后再用新账号push一次。Congratulations!
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484574122.png" width="2560" height="1276" align="center">

总结下所有的操作过程：    

```
pod trunk register your_old_mail_account //登录你的旧账号
pod trunk add-owner your_pod_name your_new_mail_account //增加你的新账号到Pod    
pod trunk register your_new_mail_account //登录你的新账号
pod trunk push
```
加上蛋疼的网络整个过程也不到20分钟，一个优秀的Error信息给使用者提升的效率是很乐观的。举个例子，如果这次的报错信息仅仅是“Request denied”，这类意义不大的提示，此次解决问题所花的时间可能会成倍往上涨。因而在软件开发过程中，尤其是SDK开发者，如何提供一套科学的报错信息也是至关重要的。