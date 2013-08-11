---
date: 2013-08-12
layout: post
title: Nodejs中的async的使用
description: 关于async中auto方法的简要说明
permalink: '/2013/08/12/how_to_use_async_in_nodejs.html'
categories:
- nodejs
tags:
- nodejs

---

# Nodejs中的async的使用

Nodejs中最最感觉不舒服的是因为全异步引起的代码层级变的更大而导致阅读更加麻烦。
所以解决因为nodejs的这个问题，我按照前人的提示选择了async的框架


其实这个框架里我更多的是使用async的auto方法
因为这个方法可以并行，串行，按照我写的规则去执行一些异步方法。

比如可以这样
async.auto({
     task1:function(cb, paramData){
     },
     task2:function(cb, paramData){
     },
     task3:['task1','task2',function(cb, paramData){
     }
     task4:['task3',function(cb, paramData){
     }
},function(err, result){
});

这个流程最后跑的是task1和task2并发执行，执行完后，task3执行，然后再task4执行，最终完成后执行最下面的function函数，如果中途的cb中的参数不为null，则直接退出上面的task流程，直接执行最后的function函数。