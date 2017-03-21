---
layout:		post
title:			"RuntimeTrick-如何判断一个子类是否复写了父类的某个方法"
subtitle:		“开拓思维总结了四个方法"
date:			2017-3-21 23:00:00
author:		"Parsifal"
header-img:	"img/postresources/coding.jpg"
catalog:     true
abstract:    "-昨天在仿写[网易BaymaxDemo](https://github.com/ParsifalC/NetEaseBaymaxDemo)的时候，遇到一个需求——要滤掉那些已经复写了`forwardInvocation:`方法自己实现消息转发的类。由于`NSObject`对于`forwardInvocation:`默认的处理是调用`unrecognizeSelector:`抛出异常，这样我们就不能使用`responseToSelector:`判断了，因为这个方法会追溯到父类的方法，也就是说他最终会找到`NSObject`的实现，从而返回YES。但这显然不是我需要的。于是展开脑洞，发散思维，想了以下四种方案。 
				"
tags:
- 侃侃技术
- iOS
- Objective-C
- Runtime

---
## 目录    
{:.no_toc}    
1.    
{:toc}

## 扯扯闲话
昨天在仿写[网易BaymaxDemo](https://github.com/ParsifalC/NetEaseBaymaxDemo)的时候，遇到一个需求——要滤掉那些已经复写了`forwardInvocation:`方法自己实现消息转发的类。由于`NSObject`对于`forwardInvocation:`默认的处理是调用`unrecognizeSelector:`抛出异常，这样我们就不能使用`responseToSelector:`判断了，因为这个方法会追溯到父类的方法，也就是说他最终会找到`NSObject`的实现，从而返回YES。但这显然不是我需要的。于是展开脑洞，发散思维，想了以下四种方案。

## 脑洞大开的四种思路
- 最简单高效的方法：直接判断IMP是否相等


```swift
- (BOOL)aIsMethodOverride:(Class)cls selector:(SEL)sel {
    IMP clsIMP = class_getMethodImplementation(cls, sel);
    IMP superClsIMP = class_getMethodImplementation([cls superclass], sel);
    return clsIMP != superClsIMP;
}
```

- 稍微有点麻烦的方法：判断Method是否相等，缺点在于要分别写实例方法和类方法两套逻辑

```swift
//以实例方法为例
- (BOOL)bIsMethodOverride:(Class)cls selector:(SEL)sel {
    Method clsMethod = class_getInstanceMethod(cls, sel);
    Method superClsMethod = class_getInstanceMethod([cls superclass], sel);
    return clsMethod != superClsMethod;
}
```

- 最低效的方法：把子类方法列表拷出来，遍历后一一比较

```swift
- (BOOL)cIsMethodOverride:(Class)cls selector:(SEL)sel {
    u_int count = 0;
    Method *methods = class_copyMethodList(cls, &count);
    BOOL isOverride = NO;
    
    for (int i = 0; i < count; i++) {
        Method m = methods[i];
        if (sel == method_getName(m)) {
            isOverride = YES;
            break;
        }
    }
    
    if (methods != NULL) {
        free(methods);
        methods = NULL;
    }
    
    return isOverride;
}
```

- 最常见但有坑的方法：这个方法经常用于做Method Swizzle里判断子类是否实现了父类方法，以避免父类方法被误伤，用于方法交叉这个场景的时候，它是合适的。但如果仅仅是判断的话，这个方法就显得有点坑了，因为你加进去的方法是无法移除的，在obcj1.0之后runtime并没有提供这个方法了。

```swift
//这种方法有很大弊端 仅当开拓思维的参考 因为方法添加后无法移除
- (BOOL)dIsMethodOverride:(Class)cls selector:(SEL)sel {
    BOOL didAddMethod = class_addMethod(cls, sel, NULL, "v@:");
    return !didAddMethod;
}
```

## 参考资料
- [我是工程源码](https://github.com/ParsifalC/DemoRepos/tree/master/IsMethodOverrideDemo)
