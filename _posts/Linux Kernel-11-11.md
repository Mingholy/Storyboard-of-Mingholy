---
title:  "Linux内核 11-11"
date:   2015-11-11 17:00:00
cover: "2.jpg"
tags:
    - Linux
    - OS
category: "Linux"
---

`光棍节快乐`

## 缓冲区

为了防止意外刺激引发读入垃圾数据，需要使用uptodate缓冲块，从块读出后，更新uptodate位。

同一缓冲块被多个进程共享，若某一时刻，一个缓冲块已由一个进程挂起，此时若新来一个进程，它并不知道自己是不是第一次读。此时需要判断bufferhead中的uptodate。

写的情形：如果是第一次申请写一个块，需要不需要确认uptodate？

*  假设该块是一个新文件，它对应的硬盘块也是垃圾数据。操作系统已经明确该文件是新的，新建i节点，并且是写操作，此时不需要确认uptodate位，因为新建操作就是用垃圾数据覆盖垃圾数据（内存中实际上是被清零了的。）当进程--缓冲区-硬盘块建立联系之后，立即置uptodate为1。
*  假设该缓冲块不是第一次写，而且可能也写不满一个缓冲块。那么就有一种情况，是后来的进程没有写满而该缓冲块还带着以前进程的数据，就产生了错误。

**uptodate:保护数据正确性。**

>当一个进程写完了一个块，另一个进程马上又更改了这个块，何如？
重点：两个进程必须是共享该缓冲块。操作系统认定两个进程一定是对这个块具有同等的操作权限的。

>首先最重要的是设备号块号，缓冲区的核心是“复用”。一个使缓冲区失去意义的应用场景是，不存在复用

补充：

*  进程0第一次切换进程1，从0特权级切到3特权级。当时进程0是可中断等待状态，进程1是运行态。通过任务切换方式切换。为什么能够判断是切到3特权级？因为所有的关于段的信息，都写在tss里，它里面存储的就是恢复现场时寄存器里应该有的**关于段的**数据。而特权级是基于段的，所以tss信息确定了之后，特权级也就确定了。
*  第二次是读引导块，进程0是可中断挂起状态，进程1是不可中断挂起。c=-1屏蔽了while中修改c和next的操作，使得switch_to参数强行是0，即切换到进程0，特殊之处在于此时*并没有运行态的进程*。
*  第三次是进程0，切换到0。进程1是不可中断等待状态，因为它正在等待读盘，读完盘中断才能切回来恢复。switch_to中比较发现要切换的目标进程正式进程0本身。于是直接跳过中间好几行代码。直接到“1：”。

## 进程的加载与退出
创建进程， 重要的东西是task_struct
fork=>copy_proc

>进程0创建进程1以后代码自动就填充了。move_to_user_mode就地翻身，它的代码是写在内核中的，是系统程序员设计好的。

一般的用户进程，开始执行后，首先就去fork来一套，创建进程框架。
某个用户进程，一定是由某个父进程创建的。linux为例，用户执行某一程序就是由shell创建的。windows gui只是封装了这一操作。（暂时这么理解。）

>创建进程之后，该父进程还要加载它。fork的时候还是3特权级，fork之后去往int80走系统调用进入内核态执行0特权级代码，中断回来之后又回到3特权级。
而子进程可以全程执行父进程的代码，它们可以一直共享资源。

`_syscall3`走3特权级

根据文件系统分配代码和数据的的位置。
