---
title: 涨姿势-NSCache
date: '2013-05-30'
description:
permalink: '/2013/05/30/about_nscache.html'
categories:

---

# NSCache
他的用法很简单，基本等效于NSMutableDictionary，只不过他不保证cache里的对象永远存在，也许在低内存警告时会释放掉部分对象，所以如果你的程序里如果有可以重复创建的对象需要保存时，可以考虑使用这个类。

以下是转载来的。
NSCache 是iOS4以后引入的一个方便的缓存某些object的类。它的使用方法与NSMutableDictionary很相似，但是他会在内存吃紧的时候自动释放某些object。而且不用考虑线程安全的问题。具体的可以参见官方文档的描述。以下是stackoverflow里面一个比较不错的应用例子：

You use it the same way you would use NSMutableDictionary. The difference is that whenNSCache detects excessive memory pressure (i.e. it's caching too many values) it will release some of those values to make room.

If you can recreate those values at runtime (by downloading from the Internet, by doing calculations, whatever) then NSCache may suit your needs. If the data cannot be recreated (e.g. it's user input, it is time-sensitive, etc.) then you should not store it in an NSCache because it will be destroyed there.


```
You use it the same way you would use NSMutableDictionary. The difference is that whenNSCache detects excessive memory pressure (i.e. it's caching too many values) it will release some of those values to make room.

If you can recreate those values at runtime (by downloading from the Internet, by doing calculations, whatever) then NSCache may suit your needs. If the data cannot be recreated (e.g. it's user input, it is time-sensitive, etc.) then you should not store it in an NSCache because it will be destroyed there.

Example, not taking thread safety into account:

// Your cache should have a lifetime beyond the method or handful of methods
// that use it. For example, you could make it a field of your application
// delegate, or of your view controller, or something like that. Up to you.
NSCache *myCache = ...;
NSAssert(myCache != nil, @"cache object is missing");

// Try to get the existing object out of the cache, if it's there.
Widget *myWidget = [myCache objectForKey: @"Important Widget"];
if (!myWidget) {
    // It's not in the cache yet, or has been removed. We have to
    // create it. Presumably, creation is an expensive operation,
    // which is why we cache the results. If creation is cheap, we
    // probably don't need to bother caching it. That's a design
    // decision you'll have to make yourself.
    myWidget = [[[Widget alloc] initExpensively] autorelease];

    // Put it in the cache. It will stay there as long as the OS
    // has room for it. It may be removed at any time, however,
    // at which point we'll have to create it again on next use.
    [myCache setObject: myWidget forKey: @"Important Widget"];
}

// myWidget should exist now either way. Use it here.
if (myWidget) {
    [myWidget runOrWhatever];
}
```



