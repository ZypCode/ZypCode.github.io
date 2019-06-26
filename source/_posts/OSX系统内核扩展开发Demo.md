---
title: OSX系统内核扩展开发Demo
date: 2019-06-19 10:38:37
categories:
- 教程
tags:
- osx系统内核
---

可以通过Mac系统的内核扩展做很多事情，这里总结些能够实现的功能。

## 文件访问拦截

[官方文档](https://developer.apple.com/library/archive/technotes/tn2127/_index.html#//apple_ref/doc/uid/DTS10003591-CH1-SUBSECTION11)

[官方sample](https://developer.apple.com/library/archive/samplecode/KauthORama/Introduction/Intro.html#//apple_ref/doc/uid/DTS10003633)

使用苹果在10.4后引入的Kauth（Kernel Authorization）实现。

Kauth里的Vnode Scope，会回调所有的文件系统操作给KEXT中注册的回调函数，包括读写、执行、删除等，并且可以由KEXT决定是否拒绝访问。

**重要函数**
```c
kauth_listen_scope 注册一个监听器

#define KAUTH_RESULT_ALLOW      (1)
#define KAUTH_RESULT_DENY       (2)
#define KAUTH_RESULT_DEFER      (3)
3个返回值，用来区分是否拒绝访问。
```


## 网络访问拦截

[官方文档](https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/NKEConceptual/socket_nke/socket_nke.html#//apple_ref/doc/uid/TP40001858-CH228-SW1)

[官方sample](https://developer.apple.com/library/archive/samplecode/tcplognke/Introduction/Intro.html#//apple_ref/doc/uid/DTS10003669)

使用苹果提供的NKEs(Network kernel extensions)实现。

使用NKE中的socket filter实现网络访问拦截。

**重要函数**
```c
sflt_register 注册一个filter

对于TCP，设置2个回调函数返回值为非0
sf_connect_in_func		sf_connect_in;
sf_connect_out_func		sf_connect_out;
```