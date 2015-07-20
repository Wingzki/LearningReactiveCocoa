#深入了解 RecativeCocoa(Objective-C)
作者：Wingzki Github:<https://github.com/Wingzki>

##什么是RecativeCocoa

RecativeCocoa是基于Cocoa框架的函数响应编程框架，为Cocoa添加完整的函数式和响应式编程支持。
Github地址:<https://github.com/ReactiveCocoa/ReactiveCocoa>


##为什么要使用RecativeCocoa

* 函数式编程可以有效提高代码的可复用性以及功能的组织性
* 响应式编程可以让你从由复杂的数据变动导致的混乱的页面刷新回调中解脱出来
* 统一了Cocoa下的各种通信与回调手段，如：`KVO` `Notification` `Target-Action` `Delegate`

##基本感念

* Event `用于通讯的核心手段`
* Signal `Event传递的管道`
* Subscriber `Event的接收者`

##基本类说明

###RACStream

RACStream的原文描述:

 ```
 An abstract class representing any stream of values
 ```
 
`RACStream`是一组被流化的值得抽象，类名也确实可以说明这一点（与`RACSignal`比起来要传神的多）。`Stream`表示是随着时间改变的值，貌似中文是“流”，它是整个`ReactiveCocoa`的基石，后面要说到的`RACSignal`和`RACSequence`就是继承自`RACStream`。我觉得简单一些的理解就是，`Stream`是一个值的队列的一个切面，排在前面的值先出，排在后面的值后出，但所有要出去的值都要经过这个切面，之后我们所有要做的事情也是针对这个切面进行。

###RACSignal `Push Driven`

`RACSignal`继承自`RACStream`，所以它也是一个`Stream`。但是它是`Push Driven`的，就是说`Siganl`每收到一个值就推一个值出去。之前一直对`Signal`这个叫法比较困惑，觉得既然有了`RACStream`，为什么还要在弄个`RACSignal`出来，我想作者估计就是想在`RACStream`的基础上分别做`Push Driven`和`Pull Driven`的两种`Stream`出来。不过目前来看，貌似作者在`Swift`版的`RecativeCocoa`中放弃了这种设计，只剩下一个`Siganl`了。

###RACSequence `Pull Driven`

`RACSequence`同样继承自`RACStream`，与`RACSignal`不同的是，`RACSequence`是`Pull Driven`的，就是说它的订阅者每需要一个值，它才“吐”一个值出去。

###RACSubscriber

##进阶类说明

* RACSubject
* RACCommand
* RACDisposable
* RACScheduler
* RACTuple
* RACChannel

##基本操作

* subscribe
* map
* filter

##进阶操作

* flatten

##各个方法的使用场景
##使用中的坑及解决途径




