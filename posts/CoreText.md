---
title: Core Text
date: '2013-10-01'
description: core text
permalink: '/2013/10/01/coretext.html'
categories:
- coretext
tags:
- coretext

---
# Core Text

1. Here you need to create a path which bounds the area where you will be drawing text. Core Text on the Mac supports different shapes like rectangles and circles, but for the moment iOS supports only rectangular shape for drawing with Core Text. In this simple example, you’ll use the entire view bounds as the rectangle where you will be drawing by creating a CGPath reference from self.bounds.