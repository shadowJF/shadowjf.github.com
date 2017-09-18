---
layout: post_layout
title: 线程数限制
time: 2017年09月18日 星期一
location: 北京
pulished: true
tag: Issue
excerpt_separator: "----"
---

　　最近测试环境工程貌似不太稳定，查看机器人服务的日志，发现其中有一条之前没有出过的错，Unhandled exception java.lang.OutOfMemoryError: unable to create new native thread，咋看之下不是内存溢出么，难道内存不够了，然而并没有那么简单，下面就记录下这个问题的解决过程。

----

# 内存溢出 #

　　一看到OutOfMemoryError，我的第一反应就是内存溢出，然后准备叫同事给机器加内存。然而事情似乎并没有这么简单，我用free命令查看了一下机器内存，发现妥妥地还剩2个G左右的内存，怎么会内存不够呢？

　　在仔细瞅一眼报的错，后面跟了个unable to create new native thread，百度一下，发现根本不是内存不足，而是无法创建更多的线程了。也就是说线程数达到上限了。

　　那么，为什么线程数会达到上限呢，在这篇文章中说的很清楚：[http://sesame.iteye.com/blog/622670](http://sesame.iteye.com/blog/622670)

　　简单来说就是，对于java程序，我们能创建的线程数是有限的，当我们创建的线程数超过这个限制，就会报错，而能创建的线程数的个数，可以由如下公式推出：

	(MaxProcessMemory - JVMMemory - ReservedOsMemory) / (ThreadStackSize) = Number of threads 

　　其中：

	MaxProcessMemory：进程的最大寻址空间，不同系统的最大寻址空间不一致，具体可以参考http://www.cnblogs.com/svennee/p/4331549.html中的说明
	JVMMemory：JVM内存，（堆+永久区）-Xmx大小 (应该是实际分配大小)
	ReservedOsMemory：操作系统预留内存
	ThreadStackSize：线程栈的大小，JVM参数Xss指定

　　可见，当你把JVM内存设的越大，反而能创建的线程数还越少，或者线程栈设的越大，能创建的线程数也会越少，然后我看了眼我们的工程，Xmx Xms Xss都是用的默认值，那我先改小Xss好了，我在JVM参数里加上-Xss256k，然后启动程序，发现果然好使了，也没有报错了。（然而事实证明这只是侥幸没出问题）

　　正当我开心之时，过了一会，发现又报了相同的错，好吧，既然Java程序能创建的线程数我已经修改了，那如果还是会超线程数，那是不是只能是线程数超过操作系统的线程数限制了呢？

　　于是机智的我上网查了一下linux操作系统对线程数的限制，发现确实有问题，参考如下文章：[http://www.cnblogs.com/nizuimeiabc1/p/5593637.html](http://www.cnblogs.com/nizuimeiabc1/p/5593637.html)

　　首先，打开/etc/security/limits.d/xx-nproc.conf（这里xx应该是个数字），内容如下：

	
	*          soft    nproc     4096
	root       soft    nproc     unlimited

　　然后，* 代表其他用户的线程数量最多为4096个，而root用户的线程数量是没有限制的，说明可以系统最大线程数，但是普通用户不行，而我们的工程都是用gateway用户启动的，而其中某一个server的线程数量居然有3000多个，查看工程线程数可以用如下指令：

	pstree -p 进程号 | wc -l

　　所以再加上其他一些工程的线程，总数量很有可能就超过4096的限制了，就算你调整jvm参数，使得在单个jvm程序里可以创建的线程数增加，但是系统的限制却没有提升导致还是报错了。

　　于是果断用root用户，修改上面那个文件里的线程限制数，改为10000，然后保存退出，然后，再重启程序，ok，没问题了。

　　

　　