---
date: 2013-04-08
layout: post
title: 如何在XCode中以不同的颜色分级显示不同级别的log信息
description: XCode分级显示log信息，并且以不同颜色去显示
categories:
- ios
tags:
- log

---


转发：
原文出处：http://maxwin.me/blog/?p=124

## 安装：
XLog – Xcode log分级显示插件

下载链接http://maxwin.me/blog/wp-content/uploads/2012/05/XLog.zip
提供功能：

1. 支持log分级，Debug/Info/Warn/Error

2. 支持自定义log颜色

安装可查看readme文件，其实就是./install.sh将插件XLog.bundle复制到~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins/目录下

## 使用
引入工程中的XLogUtil.h和.m文件,也可看下列代码.
调用类似这样的代码XLog_v(@"start!");即可。（仔细查看.h文件，里面还提供其他的分级方法，如XLog_i更多的自己查看）。

```
//
//  XLog.h
//  TestMXLog
//
//  Created by WenDong Zhang on 5/9/12.
//  Copyright (c) 2012 __MyCompanyName__. All rights reserved.
//
#import <Foundation/Foundation.h>

#define XLOG_ESC_CH @"\033"
#define XLOG_LEVEL_DEBUG    @"DEBUG"
#define XLOG_LEVEL_INFO     @"INFO"
#define XLOG_LEVEL_WARN     @"WARN"
#define XLOG_LEVEL_ERROR    @"ERROR"

// colors for log level, change it as your wish
#define XLOG_COLOR_RED   XLOG_ESC_CH @"#FF0000"
#define XLOG_COLOR_GREEN XLOG_ESC_CH @"#00FF00"
#define XLOG_COLOR_BROWN  XLOG_ESC_CH @"#FFFF00"
// hard code, use 00000m for reset flag
#define XLOG_COLOR_RESET XLOG_ESC_CH @"#00000m"   


#if defined (__cplusplus)
extern "C" {
#endif

    void _XLog_print(NSString *tag, NSString *colorStr, const char *fileName, const char *funcName, unsigned line, NSString *log);
    
    void _XLog_getFileName(const char *path, char *name);
    
    BOOL _XLog_isEnable();

#if defined (__cplusplus)
}
#endif

#define XLog_log(tag, color, ...) _XLog_print(tag, color, __FILE__, __FUNCTION__, __LINE__, [NSString stringWithFormat:__VA_ARGS__])
#define XLog_d(...) XLog_log(XLOG_LEVEL_DEBUG, XLOG_COLOR_GREEN, __VA_ARGS__)
#define XLog_i(...) XLog_log(XLOG_LEVEL_INFO, XLOG_COLOR_RESET, __VA_ARGS__)
#define XLog_v(...) XLog_log(XLOG_LEVEL_WARN, XLOG_COLOR_BROWN, __VA_ARGS__)
#define XLog_e(...) XLog_log(XLOG_LEVEL_ERROR, XLOG_COLOR_RED, __VA_ARGS__)

```


```
//
//  XLog.m
//  TestMXLog
//
//  Created by WenDong Zhang on 5/9/12.
//  Copyright (c) 2012 __MyCompanyName__. All rights reserved.
//

#import "XLogUtil.h"

//static int isXLogEnable = -1;   // -1: not set, 0: disable, 1: enable

void _XLog_print(NSString *tag, NSString *colorStr, const char *fileName, const char *funcName, unsigned line, NSString *log)
{
    const char *tagStr = [tag UTF8String];
    // show filename without path
    char *file = (char *)malloc(sizeof(char) * strlen(fileName));
    _XLog_getFileName(fileName, file);
    
    if (_XLog_isEnable()) {
        printf("%s", [colorStr UTF8String]);    // log color
        printf("%s[%s]", [XLOG_ESC_CH UTF8String], tagStr); // start tag
    }
    
    printf("%s ", [[[NSDate date] description] UTF8String]);   // time
    printf("%s %s:l%u) ", file, funcName, line);    // fileName
    printf("%s", [log UTF8String]);    // log 
    
    if (_XLog_isEnable()) {
        printf("%s[/%s]", [XLOG_ESC_CH UTF8String], tagStr);    // end tag
        printf("%s", [XLOG_COLOR_RESET UTF8String]);    // reset color
    }
    printf("\n");
    
    free(file);
}

BOOL _XLog_isEnable()
{
    return YES;
//    if (isXLogEnable == -1) {   // init
//        char *xlogEnv = getenv("XLOG_FLAG");
//        if (xlogEnv && !strcmp(xlogEnv, "YES")) {
//            isXLogEnable = 1;
//        } else {
//            isXLogEnable = 0;
//        }
//    }
//
//    if (isXLogEnable == 0) {
//        return NO;
//    }
//    return YES;
}

void _XLog_getFileName(const char *path, char *name)
{
    int l = strlen(path);
    while (l-- >= 0 && path[l] != '/') {}
    strcpy(name, path + (l >= 0 ? l + 1 : 0));
}
```

