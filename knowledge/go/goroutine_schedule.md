goroutine与调度器
=============
https://studygolang.com/articles/1855

我们都知道Go语言是原生支持语言级并发的，这个并发的最小逻辑单元就是goroutine。goroutine就是Go语言提供的一种用户态线程，当然这种用户态线程是跑在内核级线程之上的。当我们创建了很多的goroutine，并且它们都是跑在同一个内核线程之上的时候，就需要一个调度器来维护这些goroutine，确保所有的goroutine都使用cpu，并且是尽可能公平的使用cpu资源。

这个调度器的原理以及实现值得我们去深入研究一下。支撑整个调度器的主要有4个重要结构，分别是M、G、P、Sched，前三个定义在runtime.h中，Sched定义在proc.c中。

- Sched结构就是调度器，它维护有存储M和G的队列以及调度器的一些状态信息等。
- M代表内核级线程，一个M就是一个线程，goroutine就是跑在M之上的；M是一个很大的结构，里面维护小对象内存cache（mcache）、当前执行的goroutine、随机数发生器等等非常多的信息。
- P全称是Processor，处理器，它的主要用途就是用来执行goroutine的，所以它也维护了一个goroutine队列，里面存储了所有需要它来执行的goroutine，这个P的角色可能有一点让人迷惑，一开始容易和M冲突，后面重点聊一下它们的关系。
- G就是goroutine实现的核心结构了，G维护了goroutine需要的栈、程序计数器以及它所在的M等信息。
理解M、P、G三者的关系对理解整个调度器非常重要，我从网络上找了一个图来说明其三者关系：


