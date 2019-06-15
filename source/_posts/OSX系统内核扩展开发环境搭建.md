---
title: OSX系统内核扩展开发环境搭建
date: 2019-06-15 07:53:36
categories:
- 教程
tags:
- osx系统内核
---
最近项目需要对Mac系统的内核扩展开发进行了调研，本文记录开发环境的搭建以及简单的示例。



##虚拟机安装Mac系统
因为内核的开发经常容易引起系统崩溃，所以最好使用虚拟机，通过快照能够方便的还原。
1. 制作镜像
	参考https://iweiyun.github.io/2018/12/21/mac-kernel-dev/
	此处有制作好的镜像在百度云：
		链接:https://pan.baidu.com/s/1QkZXaIc9m1hhx7cSDpq9bA  密码:yccj
2. 虚拟机工具安装
	这里选择使用[VMware Fusion](https://www.newasp.net/soft/462096.html)
3. 创建虚拟机



##虚拟机环境搭建
4. 安装Kernel Debug Kit
	在[苹果开发工具](https://developer.apple.com/download/more/)里下载安装，必须与系统版本一样，这里是10.14.5 (18F132)
	
5. 设置启动参数
  ```shell
	sudo nvram boot-args="-v debug=0x140 kdp_match_name=en0 kcsuffix=development pmuflags=1"
  ```
  
6. 替换development内核

   ```shell
   sudo cp /Library/Developer/KDKs/KDK_10.14_18A391.kdk/System/Library/Kernels/kernel.development /System/Library/Kernels/kernel
   sudo kextcache -invalidate /
   sudo reboot
   ```

7. 关闭System Integrity Protection
	在上一步重启后，按住Cmd+R进恢复模式，选择实用程序，选择终端，执行命令
	```shell
csrutil disable
	```
	然后再重新启动，此时会先显示一系列命令，表示配置完成。
	这里通过ifconfig查看下虚拟机ip，后续远程调试需要。
	
8. 拍摄快照
	启动完成以后拍摄快照保存，以便后续还原



##运行内核扩展示例代码
9. 下载官方提供的示例[KauthORama](https://developer.apple.com/library/archive/samplecode/KauthORama/Introduction/Intro.html#//apple_ref/doc/uid/DTS10003633)
10. 编译示例代码
首先根据需求修改代码：
```c
if ((gPrefix == NULL) || (( (vpPath != NULL)  && strprefix(vpPath, gPrefix)) || ((dvpPath != NULL) && strprefix(dvpPath, gPrefix)))) 
{
		printf("scope=" KAUTH_SCOPE_VNODE ",action=%s,uid=%ld, vp=%s, dvp=%s\n",actionStr,			        (long)kauth_cred_getuid(vfs_context_ucred(context)),
                (vpPath  != NULL) ?  vpPath : "<null>",
                (dvpPath != NULL) ? dvpPath : "<null>"
            );
            
            if(vpPath != NULL && MyStrnstr(vpPath, "hahaha", 100)!=0){
            		asm("int $3");
                printf("check vnode here!!");
                result = KAUTH_RESULT_DENY;
            }
        }
```
当操作事件中的文件路径包含hahaha时，拒绝该操作。

为了在加载的时候能找到符号还要在Info.plist中的OSBundleLibraries加上com.apple.kpi.libkern 18.6.0，后面这个版本号可以在被调试的机器上面运行kextstat看到， 在Build Setting设置Debug Information Format为DWARF with DSYM File 即在Debug也生成符号文件方便源码调试， 为了让调试器主动断下来便在模块加载的时候使用int 3主要触发断点。

11. 虚拟机加载内核扩展

将编译好的kext文件拖入虚拟机，并修改权限加载：
```shell
chown -R root:wheel KauthORama.kext
kextload KauthORama.kext
sysctl -w kern.com_example_apple_samplecode_kext_KauthORama="add com.apple.kauth.vnode /KauthTest"
```
加载好以后通过sysctl -w设置KauthORama的参数，这里设置的意思是对/KauthTest目录注册Vnode类型的监听器，这也对该目录下文件的vnode类型操作都能回调处理。

12. 功能实验

因为10步中新增了对hahaha文件的拒绝，所以可以进行实验。
在其他文件夹中生成test以及hahaha 2个文件，尝试通过mv移动到/KauthTest目录，会发现前者成功，后者失败，表示功能成功。

此时可以通过命令查看日志
```shell
log show --last=3m --predicate 'sender == "KauthORama"'
```



##真机远程调试

因为10步中加了int 3进行断点，所以12步操作hahaha文件时系统会卡住，表明程序被断下，这时可以通过真机进行远程调试。

在真机里也安装与虚拟机系统版本一样的KDK，然后通过lldb进行远程调试。
```shell
lldb /Library/Developer/KDKs/KDK_10.14.5_18F132.kdk/System/Library/Kernels/kernel.development
kdp-remote 172.16.103.128 //这里就是之前查看的虚拟机ip
```

attach上以后就能看到断点情况了，通过continue即可继续执行。

到这里内核开发的环境已经搭建完成。



## 参考文件

https://iweiyun.github.io/2018/12/21/mac-kernel-dev/

http://www.alonemonkey.com/2017/11/20/get-start-antidebug-kext/

https://developer.apple.com/library/archive/technotes/tn2127/_index.html#//apple_ref/doc/uid/DTS10003591-CH1-SUBSECTION11

http://ddeville.me/2015/08/kernel-debugging-with-lldb-and-vmware-fusion

https://blog.csdn.net/majiakun1/article/details/78030073

https://objccn.io/issue-19-2/