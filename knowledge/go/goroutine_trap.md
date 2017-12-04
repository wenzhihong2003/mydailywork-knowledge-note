Goroutine陷阱
===========
来自于  http://lanlingzi.cn/post/technical/2016/0703_goroutine/

http://witchiman.top/2017/04/24/go-goroutine/


Go在语言层面通过Goroutine与channel来支持并发编程，使并发编程看似变得异常简单，但通过最近一段时间的编码，越来越觉得简单的东西，很容易会被滥用。Java的标准库也让多线程编程变得简单，但想当初在公司定位Java的问题，发现很多的同学由于没有深入了解Java Thread的机制，Thread直接New从不管理复用，那Goroutine肯定也要面临这类的问题。

### 1 Goroutine泄漏问题

Rob Pike在2012年的Google I/O大会上所做的“Go Concurrency Patterns”的演讲上，说道过几种基础的并发模式。从一组目标中获取第一个结果就是其中之一。

```go
func First(query string, replicas ...Search) Result {  
    c := make(chan Result)
    searchReplica := func(i int) { c <- replicas[i](query) }
    for i := range replicas {
        go searchReplica(i)
    }
    return <-c
}
```

在First()函数中的结果channel是没缓存的。这意味着只有第一个goroutine返回。其他的goroutine会困在尝试发送结果的过程中，如果你有不止一个的重复时，每个调用将会泄露资源。为了避免泄露，你需要确保所有的goroutine退出。一个不错的方法是使用一个有足够保存所有缓存结果的channel。

```go
func First(query string, replicas ...Search) Result {  
    c := make(chan Result,len(replicas))
    searchReplica := func(i int) { c <- replicas[i](query) }
    for i := range replicas {
        go searchReplica(i)
    }
    return <-c
}
```

另一个不错的解决方法是使用一个有default情况的select语句和一个保存一个缓存结果的channel。default情况保证了即使当结果channel无法收到消息的情况下，goroutine也不会堵塞。

```go
func First(query string, replicas ...Search) Result {  
    c := make(chan Result,1)
    searchReplica := func(i int) { 
        select {
        case c <- replicas[i](query):
        default:
        }
    }
    for i := range replicas {
        go searchReplica(i)
    }
    return <-c
}
```

你也可以使用特殊的取消channel来终止workers。

```go
func First(query string, replicas ...Search) Result {  
    c := make(chan Result)
    done := make(chan struct{})
    defer close(done)
    searchReplica := func(i int) { 
        select {
        case c <- replicas[i](query):
        case <- done:
        }
    }
    for i := range replicas {
        go searchReplica(i)
    }
    return <-c
}
```
为何在演讲中会包含这些bug？Rob Pike仅仅是不想把演示复杂化。这么做是合理的，但对于Go新手而言，可能会直接使用类似代码，而不去思考它可能有问题。

### 2 Goroutine Race问题

Go语言支持函数中定义函数，看下一个例子：

```go
func saveRequest(request *Request) {
            ….
            go func() {
                     request.Users = []{1,2,3}
                      …
                      db.Save(request)
            }
 
}
```

很多情况下，由于程序员对goroutine了解不够深入，又由于goroutine使用很容易。为了性能，很容易把一个同步函数变成异步函数，但这违背了go”不要通过共享内存来通信，相反应该通过通信来共享内存“的原则。即上述的例子中起了一个goroutine，并修改了request指针指向的对象。即使对request只读，也可能不是安全，因为你无法保证request指针不在其它goroutine中修改。

在本质上讲，goroutine的使用会增加了函数的危险系数，尤其是函数参数传递指针时。任何一个对象的操作，如果没有加上锁，当项目比较庞大时，可能不知道这个对象是不是会引起多个goroutine竞争。

什么是goroutine race（竞争）问题？官网的文章 Introducing the Go Race Detect给出的例子如下：

```go
package main

import(
    "time"
    "fmt"
    "math/rand"
)

func main() {
    start := time.Now()
    var t *time.Timer
    t = time.AfterFunc(randomDuration(), func() {
        fmt.Println(time.Now().Sub(start))
        t.Reset(randomDuration())
    })
    time.Sleep(5 * time.Second)
}

func randomDuration() time.Duration {
    return time.Duration(rand.Int63n(1e9))
}
```

这个例子看起来没任何问题，但是实际上，time.AfterFunc是会另外启动一个goroutine来进行计时和执行func()。由于func中有对t(Timer)进行操作(t.Reset)，而主goroutine也有对t进行操作(t=time.After)。 这个时候，其实有可能会造成两个goroutine对同一个变量进行竞争的情况。

那什么才是goroutine的使用正确姿势，怎么理解“通过通信来共享内存”来避免Race问题？先看一个例子：

```go
type SimpleAccount struct{
  balance int
}

func NewSimpleAccount(balance int) *SimpleAccount {
  return &SimpleAccount{balance: balance}
}

func (acc *SimpleAccount) Deposit(amount uint) {
  acc.setBalance(acc.balance + int(amount))
}

func (acc *SimpleAccount) Withdraw(amount uint) {
  if acc.balance >= int(amount) {
    acc.setBalance(acc.balance - int(amount))
  } else {
    panic("杰克穷死")
  }
}

func (acc *SimpleAccount) Balance() int {
  return acc.balance
}

func (acc *SimpleAccount) setBalance(balance int) {
  acc.balance = balance
}

type ConcurrentAccount struct {
  account     *SimpleAccount
  deposits    chan uint
  withdrawals chan uint
  balances    chan chan int
}

func NewConcurrentAccount(amount int) *ConcurrentAccount{
  acc := &ConcurrentAccount{
    account :    &SimpleAccount{balance: amount},
    deposits:    make(chan uint),
    withdrawals: make(chan uint),
    balances:    make(chan chan int),
  }
  acc.listen()

  return acc
}

func (acc *ConcurrentAccount) Balance() int {
  ch := make(chan int)
  acc.balances <- ch
  return <-ch
}

func (acc *ConcurrentAccount) Deposit(amount uint) {
  acc.deposits <- amount
}

func (acc *ConcurrentAccount) Withdraw(amount uint) {
  acc.withdrawals <- amount
}

func (acc *ConcurrentAccount) listen() {
  go func() {
    for {
      select {
      case amnt := <-acc.deposits:
        acc.account.Deposit(amnt)
      case amnt := <-acc.withdrawals:
        acc.account.Withdraw(amnt)
      case ch := <-acc.balances:
        ch <- acc.account.Balance()
      }
    }
  }()
}
```

上面的例子，SimpleAccount所有方法，当多goroutine操作是不安全的，而通过ConcurrentAccount封装，所有处理都统一通过channel通信到listen开启的goroutine，即只有一个goroutine能操作SimpleAccount中成员变量，那也就不会发现Goroutine Race问题。

![alt](http://studygolang.qiniudn.com/141113/fe0736cc6b61fe171de14fb15520af38.jpeg)

地鼠(gopher)用小车运着一堆待加工的砖。M就可以看作图中的地鼠，P就是小车，G就是小车里装的砖。一图胜千言啊，弄清楚了它们三者的关系，下面我们就开始重点聊地鼠是如何在搬运砖块的。

启动过程

在关心绝大多数程序的内部原理的时候，我们都试图去弄明白其启动初始化过程，弄明白这个过程对后续的深入分析至关重要。在asm_amd64.s文件中的汇编代码_rt0_amd64就是整个启动过程，核心过程如下：

```
CALL	runtime·args(SB)
CALL	runtime·osinit(SB)
CALL	runtime·hashinit(SB)
CALL	runtime·schedinit(SB)

// create a new goroutine to start program
PUSHQ	$runtime·main·f(SB)		// entry
PUSHQ	$0			// arg size
CALL	runtime·newproc(SB)
POPQ	AX
POPQ	AX

// start this M
CALL	runtime·mstart(SB)
```

启动过程做了调度器初始化runtime·schedinit后，调用runtime·newproc创建出第一个goroutine，这个goroutine将执行的函数是runtime·main，这第一个goroutine也就是所谓的主goroutine。我们写的最简单的Go程序”hello，world”就是完全跑在这个goroutine里，当然任何一个Go程序的入口都是从这个goroutine开始的。最后调用的runtime·mstart就是真正的执行上一步创建的主goroutine。

启动过程中的调度器初始化runtime·schedinit函数主要根据用户设置的GOMAXPROCS值来创建一批小车(P)，不管GOMAXPROCS设置为多大，最多也只能创建256个小车(P)。这些小车(p)初始创建好后都是闲置状态，也就是还没开始使用，所以它们都放置在调度器结构(Sched)的pidle字段维护的链表中存储起来了，以备后续之需。

查看runtime·main函数可以了解到主goroutine开始执行后，做的第一件事情是创建了一个新的内核线程(地鼠M)，不过这个线程是一个特殊线程，它在整个运行期专门负责做特定的事情——系统监控(sysmon)。接下来就是进入Go程序的main函数开始Go程序的执行。

至此，Go程序就被启动起来开始运行了。一个真正干活的Go程序，一定创建有不少的goroutine，所以在Go程序开始运行后，就会向调度器添加goroutine，调度器就要负责维护好这些goroutine的正常执行。

创建goroutine(G)

在Go程序中，时常会有类似代码：

```
go do_something()
```

go关键字就是用来创建一个goroutine的，后面的函数就是这个goroutine需要执行的代码逻辑。go关键字对应到调度器的接口就是runtime·newproc。runtime·newproc干的事情很简单，就负责制造一块砖(G)，然后将这块砖(G)放入当前这个地鼠(M)的小车(P)中。

每个新的goroutine都需要有一个自己的栈，G结构的sched字段维护了栈地址以及程序计数器等信息，这是最基本的调度信息，也就是说这个goroutine放弃cpu的时候需要保存这些信息，待下次重新获得cpu的时候，需要将这些信息装载到对应的cpu寄存器中。

假设这个时候已经创建了大量的goroutne，就轮到调度器去维护这些goroutine了。

创建内核线程(M)

![alt](http://studygolang.qiniudn.com/141113/fe0736cc6b61fe171de14fb15520af38.jpeg)

Go程序中没有语言级的关键字让你去创建一个内核线程，你只能创建goroutine，内核线程只能由runtime根据实际情况去创建。runtime什么时候创建线程？以地鼠运砖图来讲，砖(G)太多了，地鼠(M)又太少了，实在忙不过来，刚好还有空闲的小车(P)没有使用，那就从别处再借些地鼠(M)过来直到把小车(p)用完为止。这里有一个地鼠(M)不够用，从别处借地鼠(M)的过程，这个过程就是创建一个内核线程(M)。创建M的接口函数是:

```
void newm(void (*fn)(void), P *p)
```

newm函数的核心行为就是调用clone系统调用创建一个内核线程，每个内核线程的开始执行位置都是runtime·mstart函数。参数p就是一辆空闲的小车(p)。

每个创建好的内核线程都从runtime·mstart函数开始执行了，它们将用分配给自己小车去搬砖了。

调度核心

newm接口只是给新创建的M分配了一个空闲的P，也就是相当于告诉借来的地鼠(M)——“接下来的日子，你将使用1号小车搬砖，记住是1号小车；待会自己到停车场拿车。”，地鼠(M)去拿小车(P)这个过程就是acquirep。runtime·mstart在进入schedule之前会给当前M装配上P，runtime·mstart函数中的代码：

```
} else if(m != &runtime·m0) {
	acquirep(m->nextp);
	m->nextp = nil;
}
schedule();
```

if分支的内容就是为当前M装配上P，nextp就是newm分配的空闲小车(P)，只是到这个时候才真正拿到手罢了。没有P，M是无法执行goroutine的，就像地鼠没有小车无法运砖一样的道理。对应acquirep的动作是releasep，把M装配的P给载掉；活干完了，地鼠需要休息了，就把小车还到停车场，然后睡觉去。

地鼠(M)拿到属于自己的小车(P)后，就进入工场开始干活了，也就是上面的schedule调用。简化schedule的代码如下：

```
static void
schedule(void)
{
	G *gp;

	gp = runqget(m->p);
	if(gp == nil)
		gp = findrunnable();

	if (m->p->runqhead != m->p->runqtail &&
		runtime·atomicload(&runtime·sched.nmspinning) == 0 &&
		runtime·atomicload(&runtime·sched.npidle) > 0)  // TODO: fast atomic
		wakep();

	execute(gp);
}
```

schedule函数被我简化了太多，主要是我不喜欢贴大段大段的代码，因此只保留主干代码了。这里涉及到4大步逻辑：

![alt](http://studygolang.qiniudn.com/141113/b0630a33ceda74e0b0a476a0466a8420.jpg)

runqget, 地鼠(M)试图从自己的小车(P)取出一块砖(G)，当然结果可能失败，也就是这个地鼠的小车已经空了，没有砖了。
findrunnable, 如果地鼠自己的小车中没有砖，那也不能闲着不干活是吧，所以地鼠就会试图跑去工场仓库取一块砖来处理；工场仓库也可能没砖啊，出现这种情况的时候，这个地鼠也没有偷懒停下干活，而是悄悄跑出去，随机盯上一个小伙伴(地鼠)，然后从它的车里试图偷一半砖到自己车里。如果多次尝试偷砖都失败了，那说明实在没有砖可搬了，这个时候地鼠就会把小车还回停车场，然后睡觉休息了。如果地鼠睡觉了，下面的过程当然都停止了，地鼠睡觉也就是线程sleep了。
wakep, 到这个过程的时候，可怜的地鼠发现自己小车里有好多砖啊，自己根本处理不过来；再回头一看停车场居然有闲置的小车，立马跑到宿舍一看，你妹，居然还有小伙伴在睡觉，直接给屁股一脚，“你妹，居然还在睡觉，老子都快累死了，赶紧起来干活，分担点工作。”，小伙伴醒了，拿上自己的小车，乖乖干活去了。有时候，可怜的地鼠跑到宿舍却发现没有在睡觉的小伙伴，于是会很失望，最后只好向工场老板说——”停车场还有闲置的车啊，我快干不动了，赶紧从别的工场借个地鼠来帮忙吧。”，最后工场老板就搞来一个新的地鼠干活了。
execute，地鼠拿着砖放入火种欢快的烧练起来。
注： “地鼠偷砖”叫work stealing，一种调度算法。

到这里，貌似整个工场都正常的运转起来了，无懈可击的样子。不对，还有一个疑点没解决啊，假设地鼠的车里有很多砖，它把一块砖放入火炉中后，何时把它取出来，放入第二块砖呢？难道要一直把第一块砖烧练好，才取出来吗？那估计后面的砖真的是等得花儿都要谢了。这里就是要真正解决goroutine的调度，上下文切换问题。

调度点

当我们翻看channel的实现代码可以发现，对channel读写操作的时候会触发调用runtime·park函数。goroutine调用park后，这个goroutine就会被设置位waiting状态，放弃cpu。被park的goroutine处于waiting状态，并且这个goroutine不在小车(P)中，如果不对其调用runtime·ready，它是永远不会再被执行的。除了channel操作外，定时器中，网络poll等都有可能park goroutine。

除了park可以放弃cpu外，调用runtime·gosched函数也可以让当前goroutine放弃cpu，但和park完全不同；gosched是将goroutine设置为runnable状态，然后放入到调度器全局等待队列（也就是上面提到的工场仓库，这下就明白为何工场仓库会有砖块(G)了吧）。

除此之外，就轮到系统调用了，有些系统调用也会触发重新调度。Go语言完全是自己封装的系统调用，所以在封装系统调用的时候，可以做不少手脚，也就是进入系统调用的时候执行entersyscall，退出后又执行exitsyscall函数。 也只有封装了entersyscall的系统调用才有可能触发重新调度，它将改变小车(P)的状态为syscall。还记一开始提到的sysmon线程吗？这个系统监控线程会扫描所有的小车(P)，发现一个小车(P)处于了syscall的状态，就知道这个小车(P)遇到了goroutine在做系统调用，于是系统监控线程就会创建一个新的地鼠(M)去把这个处于syscall的小车给抢过来，开始干活，这样这个小车中的所有砖块(G)就可以绕过之前系统调用的等待了。被抢走小车的地鼠等系统调用返回后，发现自己的车没，不能继续干活了，于是只能把执行系统调用的goroutine放回到工场仓库，自己睡觉去了。

从goroutine的调度点可以看出，调度器还是挺粗暴的，调度粒度有点过大，公平性也没有想想的那么好。总之，这个调度器还是比较简单的。

现场处理

goroutine在cpu上换入换出，不断上下文切换的时候，必须要保证的事情就是保存现场和恢复现场，保存现场就是在goroutine放弃cpu的时候，将相关寄存器的值给保存到内存中；恢复现场就是在goroutine重新获得cpu的时候，需要从内存把之前的寄存器信息全部放回到相应寄存器中去。

goroutine在主动放弃cpu的时候(park/gosched)，都会涉及到调用runtime·mcall函数，此函数也是汇编实现，主要将goroutine的栈地址和程序计数器保存到G结构的sched字段中，mcall就完成了现场保存。恢复现场的函数是runtime·gogocall，这个函数主要在execute中调用，就是在执行goroutine前，需要重新装载相应的寄存器。



