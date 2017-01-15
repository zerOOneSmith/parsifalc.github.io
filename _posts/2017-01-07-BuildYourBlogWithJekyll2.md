---
layout:		post
title:			"Jekyll搭建Blog的总结(2)"
subtitle:		"——高效定制与管理你的博客"
date:			2017-1-7 23:00:00
author:		"Parsifal"
header-img:	"img/postresources/using-jekyll.jpg"
catalog:     true
abstract:    "- 如何用Jekyll继续修饰自己的博客   
				- 如何将Github Pages与自己的域名绑定    
				- 如何用自己的域名申请域名邮箱    
				- 如何高效管理博文中的图片    
				- 如何便利地将博客与公众号同步"
tags:
- 我的博客 
- Web前端
- Jekyll 
- Github Pages 
- 总结与教程
---
## 目录    
{:.no_toc}    
1.    
{:toc}

## 扯扯闲话    
上一篇博客，总结了些自己建博客的初衷与建站的基础，这篇我想记录与分享自己维护博客时的一些小技巧与心得。本文将从以下几个方面说开来：    

- 如何用Jekyll继续修饰自己的博客   
- 如何将Github Pages与自己的域名绑定    
- 如何用自己的域名申请域名邮箱    
- 如何高效管理博文中的图片    
- 如何便利地将博客与公众号同步    
    
## 如何用Jekyll对模板博客进行小修小改
这个部分，我结合自己修改title的例子说。对于大部分非Web前端工程师来说，相信大家和我一样都是基于开源的Template或者Theme来搭建自己博客框架的。本站是基于[黄玄的开源模板](https://github.com/Huxpro/huxpro.github.io)搭建的，再次感谢他。个人很喜欢黄玄维护的这个模板，干净简洁，又十分大气，且已经集成了大部分博客需要的功能。     

<img src="http://ojg3xdx9d.bkt.clouddn.com//468c3faf033489638cc89bdd95491e66.jpg" width="392" height="400" align="center">

可我一直纠结于一点——为什么这个模板的首页左上角的Title和居中的大title是一样的！！！这部分关于博客的小修小改就是围绕它说明的。
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484118366.png" width="2550" height="830" align="center">
为了改这个东西，我浏览了一遍[Jekyll的官方文档](http://jekyllcn.com/)，便大致清楚了从哪里着手做，怎么去做。听我娓娓道来。    
**首先**，观察[Jekyll的目录结构](http://jekyllcn.com/docs/structure/)。列几个使用频繁的说：    
    
- _config.yml为配置文件，也就是我们使用模板主要需要修改的地方。    
- _includes文件夹下，是一些可重用的布局，比如Header\Footer\Nav这些
- _layouts文件夹下，是主要是文章的布局文件
- _posts这个不用多说，放MD格式文章的
- _index.html这个就是博客的首页    

**然后**，我需要改的是首页里的一个文本，大概估计会在**index.html**和**_config.yml**。    
**于是**，将整个工程文件夹拖入**Sublime**看代码。（Jekyll使用**Liquid**模板语言，支持所有标准的Liquid标签和过滤器，只是看的话，应该很容易懂。）    
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484138311.png" width="1802" height="1146" align="center">    
浏览完这两个文件的代码我得到了这些信息：1）_config.yml里有个title;2)index.html中并没有做title的相关配置，但index.html是继承于page的，title的相关配置在page中;
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484139883.png" width="958" height="78" align="center">    
以上代码（这里如果用代码块导入源代码，会自动被当做程序代码处理😂只能用图片了。如果有懂的朋友望指教）可以看出，page对于title的设置依从的规律是——有“**page.title**”就显示，没有的话显示“**site.title**”(即_config.yml中设置的值)，那么剩下的工作就简单多了。修改这部分逻辑即可。于是我在index.html中新增了一个变量“**headerTitle**”并为其赋值，修改page中的title逻辑。
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484139971.png" width="760" height="190" align="center">
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484138851.png" width="2560" height="858" align="center">    

**最后**，修改完后，cmd中运行“**jekyll serve**”，本地预览下效果（http://localhost:4000/或者http://127.0.0.1:4000/）——Perfect!这样就解决了我的纠结。从整个过程来看，并不复杂。类似的需要做其他的修改也是如此。比如，**插入某个视频到主页**，直接复制该视频分享的iframe格式代码到index.html；要为博客**新增某个组件**，到对应的文件下添加即可；甚至[**修改一个主题**](http://jekyllcn.com/docs/themes/)，也很方便。即使不是很懂前端编程，也能做些小修小补。这里十分建议[本地配置好Jekyll环境](http://jekyllcn.com/docs/quickstart/)，对于调UI太方便了。

## 如何将Github Pages与自己的域名绑定    
既然有了自己能够做主的博客，当然也会有独立域名的冲动。对于此Jekyll的支持也很棒。Github Pages的[Quick Start](https://help.github.com/articles/quick-start-setting-up-a-custom-domain/)做了很详细的说明。这里只是做个搬运工，顺便帮自己强化记忆一遍。开始之前，再简单复习一遍相关概念。详见维基百科中的解释——[域名系统](https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F#.E5.9F.9F.E5.90.8D.E8.A7.A3.E6.9E.90)
> 域名解析----通俗来说就是DNS服务器将“**izmw.me**”这类适合人类看的地址解析成“xxx.xxx.xxx.xxx”这类机器看的IP地址的过程    
> A记录----A(Address)用来指定主机名（或域名）对应的IP地址记录，这里其实就是你Github Pages的实际IP    
> CNAME记录----即别名记录，这种记录允许将多个域名映射到另外一个域名，由另一个域名提供IP地址。通常用于同时提供WWW和MAIL服务的计算机。    

这三个概念明白了，就可以继续往下走了。假设你已经拥有了自己的独立域名，按照以下步骤即可绑定你的Github Pages到你的域名下。
1. **将“username.github.io”映射到我们自己的域名** 在**根目录**下添加**CNAME**文件，注意无后缀，全大写，CNAME文件里写你想要映射的域名。如果是在Github Pages里，可以直接在**Settings**里设置**Custom Domain**，Save之后，后台会自动在根目录下生成CNAME文件。
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484122290.png" width="1578" height="922" align="center">    
2. **添加A记录和CNAME记录** 进入你的域名供应商后台，都大同小异。找到域名栏中的域名解析这个功能。添加一条A记录，再添加一条CNAME记录。这里我添加了三条A记录与一条CNAME记录。A记录中的三个IP，一个是我本地Ping出来的，另外两个是[Github Help](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider)中提到的。CNAME记录只是做了“www”的一层跳转。设置后域名与Github Pages就完成绑定了。全球DNS服务器更新可能再需要一段时间，可先设置本地DNS服务器为一些公共DNS，如8.8.8.8\114.114.114.114等。

<img src="http://ojg3xdx9d.bkt.clouddn.com//1484143774.png" width="2560" height="494" align="center">    
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484143810.png" width="1682" height="408" align="center">    

## 如何用自己的域名申请域名邮箱    
有了自己的域名，再有自己域名的邮箱会更完美。其实上面就介绍过，CNAME记录的一个功能就是可以设置邮件服务。同样也是很简单的配置方法，这里我选择的是腾讯企业邮箱。    

1. **注册登录[腾讯企业邮箱](https://exmail.qq.com/login)** 注册登录到腾讯企业邮箱后台后，便可手动添加一个域名。
2. **进入域名供应商后台设置** 这里同样以阿里云为例，进入**后台**-**域名**-**解析**，然后直接选择**新手引导设置**，再选择添加邮箱-腾讯企业邮箱即可。设置后会自动添加如下记录。手动添加一下记录也是可以的。    
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484146998.png" width="1772" height="344" align="center">
3. **添加你自己域名的邮箱** 再返回腾讯企业邮箱，即添加邮箱，可允许有一个管理员账号。
4. **绑定原有邮箱** 如果想保留旧邮箱的话，也十分简单。设置前，**请确定自己的邮箱已开启POP**。使用独立域名邮箱账号登录到腾讯企业邮箱页面。左侧那栏找到其他邮箱。然后添加自己的原有邮箱。如果是原有邮箱绑定独立域名的企业邮箱，你需要这些[服务器地址](http://service.exmail.qq.com/cgi-bin/help?id=28&no=1000585&subtype=1)。

## 如何高效管理博客中的图片    
这部分**只针对使用Alfred的Mac党**。相信很多人在做开源轮子的时候，都会加一些Overview的GIF图片，比如[这个轮子](https://github.com/ParsifalC/CPCollectionViewWheelLayout)。那这些图应该怎么处理呢？通常的一些做法是，先将图“**add-commit-push**”到这个Repo上，然后再从Github上面的Repo中，找到这张图，最后获取到这张图的URL。这些步骤，如果只是偶尔做一次的话，还能忍受。但一旦你是写博文，又是发图狂魔，你就会各种Fxxk了。    
现在，你可以告别这样的日子了。因为有和我们一样爱偷懒的猿友发明了懒人工具！请star这个Repo吧——[很实用的图片上传workflow](http://kaito-kidd.com/2016/10/19/markdown-image-alfred/)。使用方法很简单，clone这个Repo，然后双击“*.workflow”这个文件安装即可。使用方法在Repo的Readme中也写的很详细。首先需要有一个[七牛的账号](https://www.qiniu.com/)。然后修改**config.py**文件即可。如果对默认的图片插入格式不满意，可以在**processer.py**文件中自行修改。使用方法如下：    
> - 下载alfredworkflow文件，双击安装    
> - 复制本地一张图片(格式为：jpg、png、gif)，或截一张图    
> - 打开任意编辑器，按下option + command + v快捷键    
> - 自动插入上传后的图片链接    

<img src="http://ojg3xdx9d.bkt.clouddn.com//1484149983.png" width="1802" height="1146" align="center">    
<img src="http://ojg3xdx9d.bkt.clouddn.com//1484150195.png" width="1802" height="1146" align="center">    


## 如何便利地将博文与公众号同步    
这里不说明如何注册公众号了，想注册的请移步这里[微信开放平台](https://open.weixin.qq.com/)，可参考的[官方教程](https://kf.qq.com/faq/120911VrYVrA130620u2iA7n.html)。好了进入正题。微信公众号现在已经很普及，很多个人开发者也都开始运营自己的公众号，我身边甚至有些朋友用公众号来替代博客的功能。对于经常写博客的人来说，如果能将博文直接推送到公众号，我认为是不错的一个推广和运营的渠道。但麻烦就在于，公众号管理后台的排版是不支持Markdown格式的。这曾经也让许多人头疼。不过总有热心的人为大家解决这个烦恼。这里推荐一个现在使用很广的一个Chrome插件——[Markdown here](https://github.com/devmidai/Markdown-here-CSS-for-wechat),或者直接访问[商店的地址](https://chrome.google.com/webstore/detail/markdown-here/elifhakcjgalahccnjkneoccemfahfoa?hl=zh-CN)。工具使用起来十分简单，只要将Markdown格式的直接粘贴到微信公众号后台的编辑器中，点击该插件即可。插件会自动将虽有Markdown格式的文本转化为微信后台所需的格式。如果对于某些排版不满意，你可以随时编辑各种分格的格式。不过有点遗憾的是，这个插件目前不支持即改即生效，改完后需要重新粘贴运行一次插件。最后附上作者暖心做的[视频教程](http://learn.bpteach.com/open/course/1?utm_source=zhihu.com&utm_campaign=cnm-vol-01&utm_medium=social&utm_term=markdown&utm_content=wechat-layout-guide)，点赞！
   
## 参考学习的一些资料    
- [Github-Quick start: Setting up a custom domain](https://help.github.com/articles/quick-start-setting-up-a-custom-domain/)
- [知乎上关于MarkDown上插入图片的讨论](https://www.zhihu.com/question/21065229)
- [知乎上关于公众号排版的热帖](https://www.zhihu.com/question/23640203)   