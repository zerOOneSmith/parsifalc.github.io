---
layout:		post
title:			"Swift的动态性的那些事儿"
subtitle:		"记录Swift中动态性使用的Tips"
date:			2017-2-25 14:00:00
author:		"Parsifal"
header-img:	"img/postresources/notebook.jpg"
catalog:     true
abstract:    "Swift是非常强调类型安全的一款强类型语言，苹果也十分推崇我们在Swift中尽可能的使用静态类型。但`Cocoa`框架最初是基于Objective-C开发的，因而我们在使用Swift开发iOS项目时候，难免会做一些动态性的东西。比如内省、动态添加属性等。Swift的动态性其实也是基于Objective-C的`runtime`机制运行的。"
tags:
- 侃侃技术
- iOS
- Swift
- Objective-C
- Runtime
---
## 目录    
{:.no_toc}    
1.    
{:toc}

## 扯扯闲话
Swift是非常强调类型安全的一款强类型语言，苹果也十分推崇我们在Swift中尽可能的使用静态类型。但`Cocoa`框架最初是基于Objective-C开发的，因而我们在使用Swift开发iOS项目时候，难免会做一些动态性的东西。比如内省、动态添加属性等。Swift的动态性其实也是基于Objective-C的`runtime`机制运行的。本文对于目前自己学习Swift过程中遇到的动态性使用做个小总结，另外再简单谈谈Swift的可选链式调用(`Optional Chaining`)来避免运行时的错误。

## 内省/自省 (Introspection)
内省是我们开发中经常需要做的一件事。比如判断某个`id`类型的对象是否是某个类？某个`delegate`是否实现了某协议方法？这些在使用Swift开发中同样也会遇到。
### 判断某个实例是否属于某个类型
这在Objective-C中会使用[`isKindOfClass`](https://developer.apple.com/reference/objectivec/nsobjectprotocol/1418511-iskind#)和[`isMemberOfClass`](https://developer.apple.com/reference/objectivec/nsobjectprotocol/1418766-ismember)做判断。Swift(3.0)里同样提供了这两个方法。


```swift
func isKind(of aClass: AnyClass) -> Bool
func isMember(of aClass: AnyClass) -> Bool
```

细心的朋友可以发现，这两个方法其实是基于`NSObjectProtocol`定义的。所以能调用这两个方法的必须是实现了`NSObjectProtocol`协议的，比如Objective-C里的`NSObject`及其子类。同样的，如果在Swift中需要调用这两个方法，也必须是属于`AnyObject`类型的。[关于`AnyObject`的可以查看喵大ONEVCAT的这篇博文](http://swifter.tips/any-anyobject/)。那么这就引申出一个问题了：Swift的一大特色就是不用每个类都要继承`NSObject`的，对于非`AnyObject`类型的对象，该怎么判断其类型呢？这里我目前有两个：1)Swift里提供有`is`关键字，这个可用于`AnyClass`，甚至可以是`Struct`或`Enum`，相当于`isKindOfClass`的作用；2)用`as AnyObject`关键字做个casting，之后就可以调用了。
> NSObjectProtocol就是OC里NSObject协议。


> If an object conforms to this protocol, it can be considered a first-class object. Such an object can be asked about its:1 Class, and the place of its class in the inheritance hierarchy 2 Conformance to protocols 3 Ability to respond to a particular message.The Cocoa root class, NSObject, adopts this protocol, so all objects inheriting from NSObject have the features described by this protocol.

```swift
//Football 继承于 Ball
class Ball {
    
}

class Football: Ball {
    
}

```
```swift
func test() {
	 //这里为了调用OC的自省方法，转为了AnyObject
    let football = Football() as AnyObject
    
    // 等同于 isKindOfClass
    if football.isKind(of: Football.self) {
        print("football isKindOfClass:Football")
    }
    
    if football.isKind(of: Ball.self) {
        print("football isKindOfClass:Ball")
    }
    
    // 与 isKind(of:) 类似
    // 不同的是 is 能够用于所有Swift Class
    // 而isKindOfClass()只能用于判断NSObject的子类或实现NSObjectProtocol的
    if football is Football {
        print("football is:Football")
    }
    
    if football is Ball {
        print("football is:Ball")
    }
    
    //        let ball = Ball()
    //        //这样调用会提示编译失败：ball并没有isKind(of:)方法
    //        if ball.isKind(of: Ball.sefl) {
    //
    //        }
    
    // 等同于 isMemberOfClass
    if football.isMember(of: Football.self) {
        print("football isMemberOfClass:Football")
    }
    
    if football.isMember(of: Ball.self) {
        print("football isMemberOfClass:Ball")
    }
}
```
```swift
//上面的代码输出结果
football is:Football
football is:Ball
football isKindOfClass:Football
football isKindOfClass:Ball
football isMemberOfClass:Football
```

### 判断某个对象是否实现了某个方法

这在Objective-C中会使用[`respondsToSelector:`](https://developer.apple.com/reference/objectivec/1418956-nsobject/1418583-respondstoselector?language=objc)来进行判断。与上面的`isKindOfClass:`等一样，这也是基于`NSObjectProtocol`的。

```Swift
func responds(to aSelector: Selector!) -> Bool
```
可以看到这里也有个`Selector`的类型。`Selector`其实是Objective-C里的概念，通过`@selector(method)`将一个方法转换并赋值给一个`SEL`类型，而在Swift3.0里，做法是`#selector(function)`。但在用这种方法生成一个`Selector`的时候，Xcode会提示你为该方法添加一个`@objc`的关键字。说到这里就必须提提`@objc`和`dynamic`这两个关键字了。
> `@objc`主要作用是将Swift中的类/方法/属性等暴露给Objective-C使用，如果你用 Swift写的类是继承自`NSObject`的话，Swift会默认自动为所有的非`private`的类和成员加上`@objc`关键字。这也意味着，原来`Cocoa`框架的类，都是有`@objc`的。
> 
> `dynamic`主要作用是将Swift中被标记的地方使用Objective-C动态调用运行时特性，这也意味着暴露给了Objective-C。

```swift
class Ball {
    
    @objc var name = "name"
    
    dynamic func ballFunction() {
        print(#function)
    }
    
}

class Football: Ball {
    
    @objc func footballFunction() {
        print(#function)
    }
    
}
```
```swift
func testFunction() {
    // 转为AnyObject 调用相关的自省方法
    let football = Football() as AnyObject

    if football.responds(to: #selector(getter: Football.name)) {
        print(football.name)
    }
    
    if football.responds(to: #selector(Football.ballFunction)) {
        football.ballFunction()
    }
    
    if football.responds(to: #selector(Football.footballFunction)) {
        football.footballFunction()
    }
    
}
```
```swift
//输出结果
====测试是否响应某个方法====
name
ballFunction()
footballFunction()
```

## 动态添加属性
Swift中依然可以使用Objective-C的`runtime`特性给`Extension`新增属性。所用到的两个方法和Objective-C中的类似，如下：


```swift
func objc_setAssociatedObject(_ object: Any!, _ key: UnsafeRawPointer!, _ value: Any!, _ policy: objc_AssociationPolicy)
func objc_getAssociatedObject(_ object: Any!, _ key: UnsafeRawPointer!) -> Any!
```
代码实例：


```swift
private var key: Void?

extension Ball {
    
    var color: UIColor? {
        get {
            return objc_getAssociatedObject(self, &key) as? UIColor
        }
        set {
            objc_setAssociatedObject(self, &key, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
    }
    
}
```

```swift
func testAddProperty() {
    let ball = Ball()
    ball.color = .red
    print("color of ball:\(ball.color)")
}

//输出
color of ball:Optional(UIExtendedSRGBColorSpace 1 0 0 1)
```

## 消息转发（Method Swizzling）
Objective-C中，做消息转发也是一个比较常见的黑魔法，通常会放在`+load`方法中完成方法交叉。而在Swift中，由于`load`方法不会被调用了（可参考[iOSTips](http://izmw.me/2017/02/10/iOSTips/#swift)），我们可以选择在`initialize`方法中实现方法交叉。做`Method Swizzling`是Objective-C的Runtime实现的，因而在Swift中要做这样的转发，必须满足有以下两个条件（当然，如果是CocoaTouch类，自然已经都满足了，因而都可以进行转发）：


- Swizzling的类必须继承自NSObject，否则没有`initialize`
- Swizzling的方法，必须是动态派发的，有`dynamic`标志

除了满足以上两个条件外，还必须注意到一点，必须确保方法交叉在一个`dispatch_once`中完成，因为方法交叉影响的是全局的效果，而`initialize`可能会被多次调用（这点可在官方文档中看到，若子类没有实现该方法或者在实现中调用了父类的该方法，都有可能使得它被多次调用）。由于`dispatch_once`已经在Swift3.0中无法使用了，可以[替代的方案有](http://stackoverflow.com/questions/39562887/how-to-implement-method-swizzling-swift-3-0)：1）全局的变量；2）静态的`Struct`或`Enum`或`Class`。个人更推荐第二种方法，看起来代码更干净些。实例操作如下：


```swift
class TestClass: NSObject {
    
    dynamic func originalMethod() {
        print("I'm original method")
    }
    
}
```

```swift
extension TestClass {
    
    override class func initialize() {
        //使用静态的Struct属性 确保只执行一次
        struct Inner {
            static let i: () = {
                let originalSelector = #selector(TestClass.originalMethod)
                let swizzledSelector = #selector(TestClass.swizzledMethod)
                
                let originalMethod = class_getInstanceMethod(TestClass.self, originalSelector)
                let swizzledMethod = class_getInstanceMethod(TestClass.self, swizzledSelector)
                
                //class_getInstanceMethod这个方法如果子类没有对应SEL的方法，会返回父类的方法
                //而我们需要替换的并不是其父类的方法，要保证我们替换的是子类的方法
                //我们需要通过class_addMethod做一下“保护”
                //先向要被替换的类中添加原方法SEL与swizzledIMP，这样的结果有两个：添加成功和失败
                //1.添加失败--已经有实现了 则我们只需要exchange两个实现就可以了 这时候替换的本来就是子类的方法
                //2.添加成功--子类原本并没有实现该方法 但我们已经通过class_addMethod添加了 且也已经把SEL改成SwizzledMethodIMP了 剩下的工作就是 把swizzledSEL的IMPreplace成之前拿到的originalMethod（这里其实就是其父类的方法）
                //class_addMethod这个方法，如果添加成功会返回true，失败返回false
                //当然，如果是只替换我们的自定义类，由于可以看到源码，可以直接exchange，而不用多此一举的判断
                /*
                let didAddMethod = class_addMethod(TestClass.self, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod))
                
                if didAddMethod {
                    class_replaceMethod(TestClass.self, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod))
                } else {
                    method_exchangeImplementations(originalMethod, swizzledMethod)
                }
                */
                method_exchangeImplementations(originalMethod, swizzledMethod)
            }()
        }
        let _ = Inner.i
    }
    
    func swizzledMethod() {
        //这时候调用的swizzledMethod() 其实是原方法-》originalMethod() 已经完成交叉了
        swizzledMethod()
        print("I'm swizzled method")
    }
    
}
```

```swift
func testMethodSwizzling() {
    let test = TestClass()
    test.originalMethod()
}

//输出结果
I'm original method
I'm swizzled method
```

## 可选链式调用（Optional Chaining）
//TODO:

## 参考链接
以下是学习过程中看到的博文，分享给大家，欢迎大家一起交流讨论和勘误，虚心接受指导和批评。

- [ANY 和 ANYOBJECT](http://swifter.tips/any-anyobject/)
- [hannibal-selector](https://www.bignerdranch.com/blog/hannibal-selector/)
- [objc-dynamic](http://swifter.tips/objc-dynamic/)
- [swift-objc-runtime](http://nshipster.cn/swift-objc-runtime/)
- [effective-method-swizzling-with-swift（已更新至Swift3.0）](https://www.uraimo.com/2015/10/23/effective-method-swizzling-with-swift/)
