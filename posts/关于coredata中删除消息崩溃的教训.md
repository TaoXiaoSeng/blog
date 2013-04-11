---
date: 2013-04-10
title: 关于CoreData中删除消息崩溃的教训
description: 关于coredata中relationship对于与删除object的影响
categories:
- Gor
tags:
- Gor

---

# 关于CoreData中删除记录导致崩溃的教训

由于之前没有深刻的使用过CoreData（虽然这个技术出来好多年了），只是简单的写了点demo学了下大概如何使用，以及看了一些说明简单了解了整个流程。但是由于项目需要深度使用（至少在我看来CoreData能很好降低耦合），所以工作中还是不得不碰到一些问题。比如这次就碰到。。。
## 前因
因为项目组需要提供SDK给外部使用，所以在做重构，而且做的调整很大很大，由于时间紧张，可能大家都是优先完成代码函数的重构，然后估摸着也很少对重构后的代码功能做测试，只是简单编译了下，然后简单的跑了下程序，看主要聊天功能没问题，就直接check in了。。。引起的后果是代码里可能有一些隐藏的bug。

我因为要修改session和message等很多DB相关的代码重写（虽然对CoreData了解不是那么深，但是根据封装好的接口去重写函数还是没什么大的关系的），所以必然涉及到很多创建会话，删除会话，删除消息，插入消息的功能。重构了部分功能代码后，测试过程中发现删除message，甚至到后面删除session（因为删除session内部涉及到了删除message）都会出现数据库保存操作时错误导致后面崩溃。

## 现象
当有单人聊天时，如果会话中含有消息，那么只删除一条消息也会导致崩溃，清空会话的消息也崩溃。
跟踪代码后，总是死在coredata save的时候出错。导致后面跟踪到错误，主动报错崩溃。

## 查bug过程
多次比对了自己修改前后的代码逻辑，也查看了之前修改前的版本也存在此问题，感觉不应该是我的代码引起的，所以开始往前翻svn log记录，慢慢的找到一个同事merge了一个比较大的功能进来，并且修改比较大，改动了数据库表信息。所以关注的点开始转向数据表，前前后后花了好几个小时，也添加删除了一些代码去测试，最终还是没有找到真正的原因。这个时候觉得还是要回去翻翻官方文档比较靠谱，因为一下子也不知道到底是哪块出问题，也不知道直接找哪块比较靠谱点。晚上回来后开始翻翻core data programming guide，最后找到了一段关于Relationship Delete Rules的。这里摘抄如下

```
Relationship Delete RulesA relationship's delete rule specifies what should happen if an attempt is made to delete the source object. Note the phrasing in the previous sentence—"if an attempt is made...". If a relationship's delete rule is set to Deny, it is possible that the source object will not be deleted. Consider again a department's employees relationship, and the effect that the different delete rules have.DenyIf there is at least one object at the relationship destination, then the source object cannot be deleted.For example, if you want to remove a department, you must ensure that all the employees in that department are first transferred elsewhere (or fired!) otherwise the department cannot be deleted.NullifySet the inverse relationship for objects at the destination to null.For example, if you delete a department, set the department for all the current members to null. This only makes sense if the department relationship for an employee is optional, or if you ensure that you set a new department for each of the employees before the next save operation.CascadeDelete the objects at the destination of the relationship.For example, if you delete a department, fire all the employees in that department at the same time.No ActionDo nothing to the object at the destination of the relationship.For example, if you delete a department, leave all the employees as they are, even if they still believe they belong to that department.
```
Deny：要求只要destination relationship有一个没有删除，那么source object就不能被删除。

nullify：但我们的表里不是用这个规则，用的是下面的nullify。这里很明确的说明了Set the inverse relationship for objects at the destination to null（我的理解是当你删除一个对象A时，那么他的relationship对象的指向A的relationship值会被设置为null）。

Cascade：级联删除，这个就不多说了

No Action: 删除A，不影响B对A的relationship，B中的relationship原来是什么值就还是什么值。

他给的example非常的明白。一个部门里有好多employee，所以一个部门和employee的关系是一对多的relationship，同时一个employee和部门的关系是一对一的relationship（架设一对一，当然一对多也可能），那么当你要删除一个部门的对象是，那么那个employee就没有部门了，那么***他与部门的relationship就要设置为null***或者你给他再找个新部门设置进去。如果你不给他找新部门，这就要求employee对部门的relationship的属性是optional，也就是说employee可以对部门的relationship值为null。

理解了这个之后，再看了看数据表中关于message对session的optional关系，确实没有设置为optional.修改后，编译运行，测试通过。到此终于解决了这个问题。

看样子回头还是要好好的看下coredata官方文档，否则一些细节出问题，真的很不好查找。


















