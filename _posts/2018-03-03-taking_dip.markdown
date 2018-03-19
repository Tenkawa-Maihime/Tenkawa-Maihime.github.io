---
layout:     post
title:      "浅谈依赖倒置"
date:       2018-03-03
author:     "Hime"
header-img: "img/article-bg.JPG"
catalog: true
tags:
    - 设计模式
---

>**依赖反转原则（Dependency Inversion Principle，DIP）**
>1. 高层模块不应该依赖于底层模块，两者都应该依赖于抽象。
>2. 抽象不应该依赖于实现，实现应该依赖于抽象。

依赖倒置（反转）是一种设计原则，通过依赖倒置我们可以解耦模块，让系统变得更加灵活、可靠。为了遵循这个原则，我们有了控制反转（Inversion of Control，IoC），它是一种设计模式，而控制反转有多种实现，依赖注入（Dependency Injection，DI）是其中之一。

### 插座和插头

我们有一个插座（顶层）和一个吹风筒（底层）。插座装在墙上，专门适配吹风筒的插头（顶层依赖底层）。一但吹风筒坏了换一个新的，新的插头不适配原先旧的插座，那必须将插座拆掉更换，这显然不科学。

因此， 我们定义一个接口（协议）——两脚扁型插头。我们让插座接受（可以插）一个两脚扁型的插头。那么，不管是哪个厂家的吹风筒，只要实现了这个接口，就可以插上去，这样子每次换新的吹风筒我们就不必更换插座了。

### Code

我们现在有一个取消服务（顶层）以及两种订单（底层）。我们将订单传入服务里并在服务中取消该订单。

那么问题来了，随着订单类型的不断增加，我们的服务也要做出相对应的调整，到了后期会有一大堆的if判断。这是因为我们的顶层依赖着底层导致的。
```
class Carpool {
    func cancel() {
        print("cancel carpool")
    }
}

class Taxi {
    func cancel() {
        print("cancel taxi")
    }
}

class CancelService {

    let order: AnyObject
   
    init(order: AnyObject) {
        self.order = order
    }
   
    func cancel() {
        if let carpool = order as? Carpool {
            carpool.cancel()
        } else if let taxi = order as? Taxi {
            taxi.cancel()
        }
    }
   
}

let service = CancelService(order: Taxi())
service.cancel()
```

我们来重构一下：

定义一个顶层接口，让底层的订单实现它，而顶层的服务接受一个实现了该协议的订单。这样一来，不管底层新增多少订单，取消操作如何变化，顶层都不会受到影响。因为底层依赖着一个顶层的接口，依赖的关系倒置了。

```
protocol Cancel {
    func cancel()
}

class Carpool: Cancel {
    func cancel() {
        print("cancel carpool")
    }
}

class Taxi: Cancel {
    func cancel() {
        print("cancel taxi")
    }
}

class CancelService {
   
    let order: Cancel
   
    init(order: Cancel) {
        self.order = order
    }
   
    func cancel() {
        self.order.cancel()
    }
   
}

let service = CancelService(order: Taxi())
service.cancel()
```
