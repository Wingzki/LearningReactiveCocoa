#深入了解 RecativeCocoa(Objective-C)
作者：Wingzki Github:<https://github.com/Wingzki>

##什么是RecativeCocoa

RecativeCocoa是基于Cocoa框架的函数响应编程框架，为Cocoa添加完整的函数式和响应式编程支持。
Github地址:<https://github.com/ReactiveCocoa/ReactiveCocoa>


##为什么要使用RecativeCocoa

* RecativeCocoa可以有效提高代码的可复用性以及稳定性（前提是正确使用，使用不当很容易把自己坑了）
* 响应式编程可以让你从由复杂的数据变动导致的混乱的页面刷新回调中解脱出来
* RecativeCocoa统一了Cocoa下的各种通信与回调手段，如：`KVO` `Notification` `Target-Action` `Delegate`

当你有能力`正确使用``RecativeCocoa`后，你会发世界变得美好了，代码变得整洁了，码代码的效率提高了，总的来说离加薪不远了。但是`不正确的使用`会给自己、队友以及公司带来不必要的麻烦，所以请一定认真学习并`正确使用`,`正确使用`,`正确使用`(重要的事说三遍)RecativeCocoa。

##基本概念

* Event `用于通讯的核心手段`
* Signal `Event传递的管道`
* Subscriber `Event的接收者`
* Side Effect

##基本说明

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

`RACSubscriber`是一个`Protocal`，在RAC中有很多类实现了这个协议。所有实现这个协议的类便具有了订阅并处理`Siganl`的能力。使用不用的方法可以处理`Siganl`不同的Event：`next``complete``error`。可以这样理解，当一个`Siganl`没有订阅者的时候，这个`Siganl`就并没有什么哔用。

###RACDisposable

`Siganl`被订阅后会返回一个RACDisposable的实例，这个实例可以用于终止信号并做一些额外的清理工作。

###RACCommand

`RACCommand`可以被理解为RAC世界中的`IBAction`，用于表示一个交互命令。我们可以用它来封装一个点击事件的处理逻辑。

##进阶说明

###RACDynamicSignal

如果看`RACSignal`的实现代码，你会发现当你订阅一个`RACSignal`的时候其实什么都没有发生。真的，确实是什么都没有发生。代码如下：

```
- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
	NSCAssert(NO, @"This method must be overridden by subclasses");
	return nil;
}
```

事实上，在一个信号被最初产生的时候，真正被实例化的是`RACSignal`的子类`RACDynamicSignal `。

`RACDynamicSignal`持有一个名为didSubscribe的Block，这个block在我们调用

```
+ (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe;
```
方法时被赋值，然后在Signal订阅时被执行。同时`RACDynamicSignal`覆盖了`RACSignal`的`subscribe`方法，重新定义了订阅操作。源码如下：

```
- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
	NSCParameterAssert(subscriber != nil);

	RACCompoundDisposable *disposable = [RACCompoundDisposable compoundDisposable];
	subscriber = [[RACPassthroughSubscriber alloc] initWithSubscriber:subscriber signal:self disposable:disposable];

	if (self.didSubscribe != NULL) {
		RACDisposable *schedulingDisposable = [RACScheduler.subscriptionScheduler schedule:^{
			RACDisposable *innerDisposable = self.didSubscribe(subscriber);
			[disposable addDisposable:innerDisposable];
		}];

		[disposable addDisposable:schedulingDisposable];
	}
	
	return disposable;
}
```

可以看到，每当subscribe方法被执行一次，didSubscribe就被执行一次。这就是为什么当有多个subscriber的时候，Side effict会被执行多次。在后面我们会提到这个问题的解决方案。

###RACSubject

RACSubject是将控制权完全交给码农的Siganl，你可以随心所欲的send XX给他，对没错，就是send XX。之前send XX不是都是对于subscriber嘛，自从有了RACSubject，世道就变了，所以愉快的接受现实吧，你会发现RAC的世界从此更加精彩。

RACSubject实现了<RACSubscriber>协议，所以它也具有send XX方法，只是行为方式不同，下面我们慢慢道来。

首先看RACSubject类扩展定义：

```
@interface RACSubject ()

// Contains all current subscribers to the receiver.
//
// This should only be used while synchronized on `self`.
@property (nonatomic, strong, readonly) NSMutableArray *subscribers;

// Contains all of the receiver's subscriptions to other signals.
@property (nonatomic, strong, readonly) RACCompoundDisposable *disposable;

// Enumerates over each of the receiver's `subscribers` and invokes `block` for
// each.
- (void)enumerateSubscribersUsingBlock:(void (^)(id<RACSubscriber> subscriber))block;

@end

```

然后看RACSubject的subscribe方法：

```
- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
	NSCParameterAssert(subscriber != nil);

	RACCompoundDisposable *disposable = [RACCompoundDisposable compoundDisposable];
	subscriber = [[RACPassthroughSubscriber alloc] initWithSubscriber:subscriber signal:self disposable:disposable];

	NSMutableArray *subscribers = self.subscribers;
	@synchronized (subscribers) {
		[subscribers addObject:subscriber];
	}
	
	return [RACDisposable disposableWithBlock:^{
		@synchronized (subscribers) {
			// Since newer subscribers are generally shorter-lived, search
			// starting from the end of the list.
			NSUInteger index = [subscribers indexOfObjectWithOptions:NSEnumerationReverse passingTest:^ BOOL (id<RACSubscriber> obj, NSUInteger index, BOOL *stop) {
				return obj == subscriber;
			}];

			if (index != NSNotFound) [subscribers removeObjectAtIndex:index];
		}
	}];
}
```
RACSubject维护了一个叫subscribers的可变数组，故名思议，这个数组是用来放subscriber的的。然后当我们订阅这个信号的时候，所有subscriber都会被保存在这个数组中。看到这个应该也就明白send
XX方法要做什么了。没错，就是将send XX转发给每一个subscriber。源码如下：

```
- (void)sendNext:(id)value {
	[self enumerateSubscribersUsingBlock:^(id<RACSubscriber> subscriber) {
		[subscriber sendNext:value];
	}];
}
```

看到这里应该就能明白`RACSubject`的强大了，源码的注释里是这么说的：

```
They're most helpful in bridging the non-RAC world to RAC.
```

要把非RAC世界的东东桥接到RAC，就要靠老子`RACSubject`了。
事实上，通过RACSubject我们可以把任意继承自NSObject的对象发送到任意多得subscriber手中。有了这样一个通道，很多复杂的调用都已变得清晰简单。

* RACReplaySubject

`RACReplaySubject`是`RACSubject`的子类，与`RACSubject`的不同之处是它维护了一个名为valuesReceived可变数组，用于储存所有收到的Value。当有一个新的Subscriber时，valuesReceived中的所有Value都会发送给新的Subscriber。
 
* RACBehaviorSubject

`RACBehaviorSubject`和`RACReplaySubject`唯一的区别就是它只会把收到的最后一个Value发送给新的Subscriber。

###RACPassthroughSubscriber

查看objc.io的这篇文章[DTrace](http://objccn.io/issue-19-4/)

###RACChannel
###RACScheduler
###RACTuple


##基本操作

* subscribe
* map
* filter

##进阶操作

* flattenMap
* flatten

##各个方法的使用场景
##使用中的坑及解决途径




