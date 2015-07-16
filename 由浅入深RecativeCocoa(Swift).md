#由浅入深 RecativeCocoa(Objective-C)
作者：Wingzki Github:<https://github.com/Wingzki>

##什么是RecativeCocoa

RecativeCocoa是基于Cocoa框架的函数响应编程框架，为Cocoa添加完整的函数式和响应式编程支持。
Github地址:<https://github.com/ReactiveCocoa/ReactiveCocoa>


##为什么要使用RecativeCocoa

* 函数式编程可以有效提高代码的可复用性以及功能的组织性
* 响应式编程可以让你从由复杂的数据变动导致的混乱的页面刷新回调中解脱出来
* 统一了Cocoa下的各种通信与回调手段`KVO` `Notification` `Action` `Delegate`

##基本感念

* Event `用于通讯的核心手段`
* Signal `Event传递的途径`
* Subscriber `Event的接收者`

##基本类说明

* RACStream
* RACSignal
* RACSubscriber

##高级类说明

* RACSubject
* RACCommand
* RACDisposable
* RACScheduler
* RACSequence
* RACTuple
* RACChannel

##基本操作

* map
* filter
* flatten

##高级操作




