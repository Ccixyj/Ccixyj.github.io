---
title: 解决 `Unresolved reference for synthetic view when layout is in library module` 问题
date: 2018-07-09 10:45:10
tags: android 
---

在使用模块化组件[CC](https://github.com/luckybilly/CC)重构时遇到了 `Unresolved reference`的问题。
错误原因是 父模块使用无法直接使用子模块的布局，kotlin-android-extensions无法解析（`com.android.tools.build:gradle` 版本3.0 以上。） 


#### 解决办法：
在父项目的`build`文件下添加
```groovy
//配置文件添加如下：
androidExtensions {
    experimental = true
}
```
[参考stackoverflow的问题](https://stackoverflow.com/questions/48378696/unresolved-reference-for-synthetic-view-when-layout-is-in-library-module)

`Kotlin Android Extensions`的高级用法可以查阅[官方文档](http://kotlinlang.org/docs/tutorials/android-plugin.html)
