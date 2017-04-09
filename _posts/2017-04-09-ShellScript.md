---
layout:		post
title:			"Where There Is A Shell,There Is A Way!"
subtitle:		"Shell学习笔记"
date:			2017-4-7 23:00:00
author:		"Parsifal"
header-img:	"img/postresources/linux-shell-scripting.png"
catalog:     true
abstract:    "-  `Shell Script`在日常工作中接触地挺多，比如CI中的打包发布等都需要`Shell Script`进行辅助完成。然而一直没有进行系统地学习，都是东凑西凑地完成一个Just Works的脚本。现在打算利用闲余时间，较为系统地去学习下这门技术，并以此文作为学习笔记记录。 
				"
tags:
- 侃侃技术
- Shell
---
## 目录    
{:.no_toc}    
1.    
{:toc}

## 扯扯闲话
`Shell Script`在日常工作中接触地挺多，比如CI中的打包发布等都需要`Shell Script`进行辅助完成。然而一直没有进行系统地学习，都是东凑西凑地完成一个Just Works的脚本。现在打算利用闲余时间，较为系统地去学习下这门技术，并以此文作为学习笔记记录。

## 概念
**Shell**是源于**Linux**系统。**Shell**在**Linux**系统中是作为用户与内核间的桥梁存在，通过**Shell**用户可以方便地操作**Linux**执行一些命令。同时**Shell**又可以理解为一种**命令式语言**和**程序设计式语言**。作为编程语言来说，**Shell**是一个解释型的语言，无编译过程，只需要一个解释器就可以运行。Mac下自带多种**Shell**解释器，如Bourne Shell（sh，Unix标配）、Bourne Again Shell（bash，Linux标配）、Z Shell（zsh，结合Oh My Zsh成为众多程序员在Mac系统里的首选Shell）等。

> Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。

**Shell Script**就是写给**Shell**执行的脚本，平常所说的**Shell**也多指代这种**Shell Script**。由于是解释型的语言，所以**Shell Script**只需要一个文本编辑器就可以编写，无需繁重的IDE支持（这里吐槽**Xcode**）。Mac下推荐**TextMate**和**Sublime Text**，都是很好用的编辑器。

## 参考资料
- [Shell 教程](http://www.runoob.com/linux/linux-shell.html)