---
date: 2013-04-22
layout: post
title: CoreData编程指南_CoreData的并发编程_章节翻译
description: 文档生成命令
permalink: '/2013/04/22/Concurrency_with_Core_Data.html'
categories:
- ios
tags:
- core data

---

# Core Data的并发（Concurrency with Core Data）
存在着一些的场景，我们需要利用后台线程或者队列来执行Core Data的一些操作，因为他们能我们带来一些好处，尤其是你需要在Core Data在做一些比较消耗时间的任务时能保证应用程序的用户交互界面能及时响应。但是如果你并发使用Core Data时，那么你需要关心，考虑到对象图(object Groups)不会进入一个冲突的状态。

如果你选择使用并发编程来操作Core Data，你还需要考虑到应用程序的环境。对于大多数的情况，AppKit和UIKit是线程不安全的；另外，OS X Cocoa bindings和controllers也都是线程不安全的 -- 如果你正在使用这些技术，多线程就会复杂了。

## 使用Thread Confinement来支持并发（Use Thread Confinement to Support Concurrency）
Core Data推荐的并发编程的模式是thread confinement（线程封闭，线程约束？）：每个线程必须拥有属于它自身的，完全私有的managed object context。

这里有两种可能的方法来采用这种模式：

1. 给每个线程创建一个managed object context，并且共享一个persistent store coordinator
2. 给每隔线程创建一个managed object context和persistent store coordinator。（这种方法提供更加强大的并发，但是同样会更加的复杂（尤其是你需要在不同的context之间进行交互）和消耗更多的内存使用）


你必须在它使用的线程中创建相应的managed context。如果你使用的NSOperation，你需要注意它的init方法是在被调用（caller）的线程里执行的。所以你不能够在queue的init方法里创建managed object context，否则你创建的context会关联到这个caller调用者的线程。与之相对应的，你应该在main方法（针对serial queue串行队列）或者start方法（针对concurent queue并发队列）里创建context。

使用thread confinement（线程封闭，线程约束？），你不应该在线程间传递managed objects或者managed object contexts。如果你需要跨线程传递一个managed object，你可以通过以下方法:
	
* 传递它的object ID（objectID）然后在接收线程里的managed object context里使用objectWithID:或者existingObjectWithID:error:去获取。相对应的managed objects必须已经保存（save）--你不能够传递一个新创建的还没有保存过的managed object的ID给另外一个context上。
* 在接收context上通过执行fetch来获取得到需要的managed object。

这些方法会在接收context上创建一个本地版本（local version）的managed object。

你能够通过NSFetchRequest的相关方法来更加方便高效的跨平台去操作数据。例如一个配置一个fetch请求来返回一些列的object IDs，当然也可以包括row data（同时更新cache的row数据）-- 这对于你想在一个后台线程里传递一些object IDs到另外的线程里是很有用的。

通常情况下你没有必要去lock managed objects或者managed object contexts。然后，如果你使用单一的被多个context共享的persistent store coordinator，并且想要在它上面执行一些操作（比如你想要添加一个新的store），或者你想要集合多个operation像一个事务一样一起在one context执行。你应该lock锁住persistent store coordinator。

## 在其他线程里使用notification通知来跟踪变动
你在一个context上对一个managed object的改动是不能自动的通知传播到另外一个不同的context上相关联对应的managed object，除非你主动重新查询（refetch）或者***re-fault(?)***这个object。如果你需要在某个线程上跟踪另外一个线程上的改动，这里有两种方法达到这个目的，两个都是通过notifications通知的方式。为了能更好的解释说明的目的，我们现在假设有两个线程A和B，假设你现在想要将B上的改动传送通知给A。

一般来说，你在线程A上注册managed object context的save通知，NSManagedObjectContextDidSaveNotification。当你接收到这个通知的时候，这个通知里的userinfo（dictionary类型）包含了在B线程里被插入，删除，更新的managed objects的数组信息。由于这些managed objects是被关联到不同的线程上，因此你不应该直接去访问这些对象。与之对应的，你应该将notification作为的一个参数来传递给[mergeChangesFromContextDidSaveNotification:](http://developer.apple.com/library/ios/#documentation/cocoa/Reference/CoreDataFramework/Classes/NSManagedObjectContext_Class/NSManagedObjectContext.html#//apple_ref/occ/instm/NSManagedObjectContext/mergeChangesFromContextDidSaveNotification:)方法（这个方法是你在用来发送相关数据给线程A）。通过使用这个方法，context才能够更加安全的合并改动。

注意变动的通知（change notification）是发送到NSManagedObjectContext的[processPendingChanges](http://developer.apple.com/library/ios/#documentation/cocoa/Reference/CoreDataFramework/Classes/NSManagedObjectContext_Class/NSManagedObjectContext.html#//apple_ref/occ/instm/NSManagedObjectContext/processPendingChanges)方法。这个方法在主线程是被绑定到应用程序的事件循环里的，这样在主线程上，processPendingChanges就能让用户在context上的每个用户事件后被自动调用。但这不适用于后台进程 -- 什么时候这个方法被调用是决定于平台（platform）和发布版本（release version），所以你不应该依赖与特定的时间。如果第二个context不是在主线程上，你应该自己在适当时刻来调用processPendingChanges方法。（你应该创建你自己概念上的后台线程上工作“循环”（周期？cycle） -- 比如在一系列的操作之后）。

## 后台查询来支持UI响应（Fetch in the Background for UI Responsiveness）
executeFetchRequest:error:方法本质上会根据硬件和工作负载来对它的行为进行一定调整。如果需要，Core Data将会创建额外的私有线程来优化查询性能。你不需要为了优化提高查询的速度的目的而特意的为此创建后台线程。当然这仍然有可能被适当使用，不管怎么说，通过后台线程或queue去查询来尽量避免你的应用程序被用户交互阻塞住。这意味着，如果一个查询很复杂，或者会返回大量的数据，那么你能通过这样的方式来控制，等到这些数据获取到的时候再展示出这些数据。

为了遵从thread confinement（线程封闭？线程约束？）的设计模式，你使用两个managed object context来关联到一个persistent store coordinator。你从一个后台线程的managed object context上来查询数据，然后传递相应的查询得到的object IDs到另外一个线程里。在第二个线程里（一般来说指的是应用程序的主线程，因为你可以得到这些数据后将他们在这个线程里展示出来），你用第二个context来fault这些传递过来的objectIDs对应的那些objects（你可以使用objectWithID:来实例化这些对象）。（这项技术仅仅对于你使用SQLite store的时候才有效，因为binary和xml stores的存储介质里的数据会在被打开的时候就都被读取到内存里了）

## 在后台线程做保存操作是容易出错的（Saving in a Background Thread is Error-prone）
异步的队列和线程都不会阻止应用程序被关闭，也就是说应用程序是不会等异步线程的退出结束之后才关闭。（具体点说就是，所有的NSThread-based线程都是“detached”（独立的？分离的？附加的？）-- 更加详细的可以查看pthread的文档 -- 而一个进程只会等到所有的not-detached线程退出后才退出）如果你在一个后台线程里执行一个保存操作，那么，这个操作可能在它完成保存操作前被杀死。如果你需要在一个后台线程里做保存操作，你必须写一些额外的代码，类似让主线程阻止应用程序在所有的保存操作完成前退出。

## 如果你不用Thread Confinement（If You Don’t Use Thread Containment）
如果你选线不使用程兼容的模式 -- 这就是说，如果你尝试在多线程间传递managed objects或者contexts等类似的操作 -- 你就必须非常消息的去锁数据（locking），这样的后果就是你可能失去了使用多线程带来的好处。你还需要关注一下问题：

* 每次你使用相关联的managed object context去操作或者访问managed objects。Core Data没有提供一个读安全（save）但是修改有危险（dangerous）的解决方案 -- 而是任何操作都是危险（dangerous）的，因为任何操作都会有为了性能考虑的缓存，都可以触发faulting。
* managed objects自身就不是线程安全的。如果你想要跨线程去操作一个managed object，你必须锁住（lock）他的context（参考查看[NSLocking](http://developer.apple.com/library/ios/documentation/cocoa/Reference/Foundation/Protocols/NSLocking_Protocol/Reference/Reference.html#//apple_ref/occ/intf/NSLocking)）

如果你在多线程间共享一个managed object context或者一个persistent store coordinator，你必须确保任何被调用的方法都是以线程安全的定义的。如果需要locking，你应该使用NSLocking方法来锁住managed object context和persistent store coordinator，而不是用你自己定义的mutexes（互斥方法）。这些NSLocking方法还帮助提供了上下文的信息给应用程序的框架 -- 换句话说，就是除了提供了mutex互斥的目的，它（NSLocking）还帮助定义了一系列Core Data操作的上下文的工作环境。

一般来说，你可以使用tryLock或者lock来锁context或者coordinator。如果你这样做了，那么框架就会保证在这之后的操作也都是线程安全。比如说，如果你为每个线程创建了一个context，但是所有的context都是指向同一个persistent store coordinator，那么Core Data会以线程安全的方式来负责访问coordinator（NSManagedObjectContext的[lock](http://developer.apple.com/library/ios/#documentation/cocoa/Reference/CoreDataFramework/Classes/NSManagedObjectContext_Class/NSManagedObjectContext.html#//apple_ref/occ/instm/NSManagedObjectContext/lock)和unlock方法支持递归））

如果你使用lock锁住了一个context（或者使用tryLock成功锁住），那么直到你调用unlock去解锁为止，你必须保持对这个context的强引用。如果你不保持强引用的话，那么在一个多线程的环境里，你可能导致死锁。