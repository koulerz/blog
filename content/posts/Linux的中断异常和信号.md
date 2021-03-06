---
title: "Linux 的中断异常和信号"
date: 2016-04-19T21:42:10+08:00
publishdate: 2016-04-19
lastmod: 2020-01-17
draft: false
tags: ["linux", "os"]
---
## 中断
**中断**是指CPU对系统发生的某个事件作出的一种反应，让CPU暂时停止当前程序的执行转而执行处理新情况的程序和执行过程。即在程序运行过程中，系统出现了一个必须由CPU立即处理的情况，此时，CPU暂时中止程序的执行转而处理这个新的情况的过程就叫做中断。

**中断向量**: 每个中断和异常是由0~255之间的一个数来标识。Intel把这个8位的无符号整数叫做一个向量。

**IRQ**: 每个能够发出中断请求的硬件设备控制器都有一条名为IRQ的输出线。所有现有的IRQ线都与一个名为可编程中断控制器的硬件电路的输入引脚相连。

**PIC**: 可编程中断控制器

**IDT**: 中断描述符表，中断描述符表是一个系统表，它与每一个中断或异常向量相联系，每一个向量在表中有相应的中断和异常处理程序的入口地址。表中的每一项对应一个中断或异常向量，每个向量由8个字节组成。因此，最多需要256*8=2048字节来存放IDT。

广义的中断包含**异步中断**和**同步中断**。

## 异步中断
**异步中断(外部中断/硬件中断)**通常被直接称为**中断(interrupt)**

CPU对其的响应完全是被动的，但是可以屏蔽掉。

异步中断可分为**可屏蔽中断**（Maskable interrupt）和**非屏蔽中断**（Nomaskable interrupt）。

所谓的**异步**指的是发生的中断不是随正在执行的指令同步发生的，是由其他硬件设备依照 CPU 时钟信号随机产生的，是不可预知的，例如用户随时在键盘上按下一个键。

## 同步中断
**同步中断(内部中断/软件中断)**通常被称为**异常(exception)**

在Intel的手册中，同步中断被称之为**异常**。

异常可分为**故障(fault)**、**陷阱(trap)**、**终止(abort)**三类。

类型 | 原因 | 异步/同步 | 返回行为 | 例子
----- | ----- | ------ | ----- |------- 
**陷阱** | 有意的异常  | 同步 |  总是返回到下一条指令 | 系统调用、信号机制 ( 通过软中断实现 )
**故障** | 潜在可恢复的错误 | 同步 |    返回到当前指令|缺页异常、除 0 错误、段错误 
**终止** |    不可恢复的错误 |   同步 |不会返回|硬件错误

## 软中断和硬中断
**软中断**是通信进程之间用来模拟硬中断的一种信号通信方式。是属于一种编程手段，也有称之为软中断通信机制。

**硬中断**一般就是指的硬件中断，也就是常说的中断，由硬件触发。


## 区别
类型 |    产生的位置 | 发生的时刻 | 时序
------------ | ------------- | ------------ | ----------
中断 | CPU外部  | 随机 |  异步
异常 | CPU内部 | 一条指令终止后 |  同步

## 信号机制
信号是异步的进程间的通讯机制，是在软件层次上对中断机制的一种模拟，在原理上，一个进程收到一个信号与处理器收到一个中断请求可以说是一样的。

信号是异步的，一个进程不必通过任何操作来等待信号的到达，事实上，进程也不知道信号到底什么时候到达。

信号是进程间通信机制中唯一的异步通信机制，可以看作是异步通知，通知接收信号的进程有哪些事情发生了。内核也可以因为内部事件而给进程发送信号，通知进程发生了某个事件。注意，信号只是用来通知某进程发生了什么事件，并不给该进程传递任何数据。

**产生信号的条件主要有**:
1. 用户在终端 按下某些键时，终端驱动程序会发送信号给前台进程，例如Ctrl-C 产生 SIGINT 信 号， Ctrl-/ 产生 SIGQUIT 信号， Ctrl-Z 产生 SIGTSTP 信号。
2. 硬件异常产生信号，这些条件由硬件检测到并通知内核，然后内核向当前进程发送适当的信号。例如当前进程执行了 除以0 的指令， CPU 的运算单元会产生异常，内核将这个异常解释为 SIGFPE 信号发送给进 程。再比如当前进程访问了非法内存地址，， MMU 会产生异常，内核将这个异常解释为 SIGSEGV 信 号发送给进程。
3. 一个进程调用kill(2) 函数可以发送信号给另一个进程。
4. 可以用kill(1)命令发送信号给某个进程，kill(1)命令也是调用kill(2)函数实现的，如果不明确指定信号则发送SIGTERM信号，该信号的默认处理动作是终止进程。
5. 当内核检测到某种软件条件发生时也可以通过信号通知进程，例如闹钟超时产生SIGALRM信号，向读端已关闭的管道写数据时产生SIGPIPE信号。

**进程对信号的处理**:
1. 忽略此信号。
2. 执行该信号的默认处理动作。
3. 提供一个信号处理函数，要求内核在处理该信号时切换到用户态执行这个处理函数，这种方式称为捕捉(Catch)一个信号。

**信号与中断的区别**:
1. 中断有优先级，而信号没有优先级，所有的信号都是平等的；
2. 信号处理程序是在用户态下运行的，而中断处理程序是在核心态下运行；
3. 中断响应是及时的，而信号响应通常都有较大的时间延迟。

**信号机制具有以下三方面的功能**:
1. 发送信号。发送信号的程序用系统调用kill()实现。
2. 预置对信号的处理方式。接收信号的程序用signal()来实现对处理方式的预置。
3. 收受信号的进程按事先的规定完成对相应事件的处理。
