---
title: more about markdown
date: '2013-06-05'
layout: post
description: more about markdown
permalink: '/2013/04/06/more-about-markdown.html'
categories:
- markdown

---

# 关于几个之前忽略的markdown的语法
1. 行末连续两个空格然后回车，可以造成文本换行

```
abcdefg“空格空格”
hijklmn
```

abcdefg  
hijklmn

2. <a name="md-anchor" id="md-anchor">段落缩进</a>
	
	段落缩进演示：	
	> 第一段
	>
	>> 第二段
	>>> 第三段
	>> 
	>> 第二段
3. 四个空格缩进是代码块：
4. 无序列表
	* 星号、减号、加号开始列表。
  		- 列表层级和缩进有关。
    		+ 和具体符号无关。
	* 返回一级列表。
	
5. 有序列表
	1. 数字和点开始有序列表。

   		1. 注意子列表的缩进位置。

      		1. 三级列表。
      		1. 编号会自动更正。

   		2. 二级列表，编号自动更正为2。

	2. 返回一级列表。


6. 上标、下标
	- Water: H<sub>2</sub>O
	- E = mc<sup>2</sup>
	
7. 等宽字体
	行内反引号嵌入代码，如: `git status` 。
	
8. 内部跳转
	<a name="md-anchor" id="md-anchor"></a>

	跳转至 [文内链接](#md-anchor) 。
	
	
	
摘抄自：[链接](http://www.worldhello.net/gotgithub/appendix/markups.html)