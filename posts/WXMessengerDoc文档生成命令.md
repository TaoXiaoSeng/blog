---
date: 2013-04-06
layout: post
title: 关于如何使用appledoc生成XCode能读取的文档的命令
description: 文档生成命令
permalink: '/2013/04/06/create-docset-used-by-xcode-the-tool-appledoc.html'
categories:
- ios
tags:
- docset

---

messenger文档生成命令

appledoc --project-name WXMessengerDoc --project-company "alibaba-inc" --company-id com.alibaba-inc.WXMessenger --output ./WXMessengerDoc --ignore ./Three20 --ignore ./ASIHttpRequest --ignore ./script --ignore ./tools --ignore ./docs --ignore ./Messenger/JSON --ignore ./Messenger/JSONKit.h --ignore ./Messenger/JSONKit.m --ignore ./Messenger/PopverView --ignore ./Messenger/RichText --ignore /Messenger/SDWebData --ignore ./Messenger/SFHFKeychainUtils --ignore ./Messenger/TouchJSON --ignore ./Messenger/weak_ref --ignore ./Messenger/zh-Hans.lproj --ignore ./Messenger/Frameworks .
