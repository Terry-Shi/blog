### Go 并发编程
#### goroutine 协程 - 比线程更加轻量级
  - go 开始一个新的协程  
	```GO
	func f(msg string) {
	    fmt.Println(msg)
	}
	
	func main(){
	    go f("goroutine") // 新的协程执行函数f
	
	    go func(msg string) { // 新的协程执行匿名函数
	        fmt.Println(msg)
	    }("going") // 小括号表示执行这个函数，"going"表示输入参数
	}
	```
  - main函数的协程结束，其他协程也立刻结束，整个程序退出。

#### channel 管道或信道／用于go协程间通信 － 类似于Java中阻塞操作Queue
  - 而Go有一个特殊的类型，通道（channel），可以通过它们发送数据在协程之间通信，可以避开所有内存共享导致的坑；通道的通信方式保证了同步性。数据通过通道：同一时间只有一个协程可以访问数据：所以不会出现数据竞争。
  - 操作符<-用于指定管道的方向，发送或接收。如果未指定方向，则为双向管道。  
	```GO
	// 以下是三种不同类型的定义，
	chan Sushi        // 可用来发送和接收Sushi类型的值  
	chan<- float64    // 仅可用来接受float64类型的值  
	<-chan int        // 仅可用来发送int类型的值   （不能接受，只能发送有什么意义?）
	// 不同管道类型之间赋值兼容性 （错误的类型赋值会在运行时抛出异常）
	var sendChan <-chan int = make(<-chan int) // receive-only type
	var receiveChan chan<- int = make(chan<- int) // send-only type
	var twoWayChan chan int = make(chan int) // receive and send type
	receiveChan = twoWayChan // OK
	twoWayChan = receiveChan // 运行时报错： cannot use receiveChan (type chan<- int) as type chan int in assignment
	
	// 初始化
	ic := make(chan int)    // 不带缓冲的int类型管道
	wc := make(chan *Work, 10)    // 带缓冲的Work类型指针管道
	// 赋值
	ic <- 3        // 往管道发送3
	work := <-wc    // 从管道接收一个指向Work类型值的指针
	```
  - 阻塞式读写: 如果channel满了写操作就会阻塞，如果channel空了读操作就会阻塞。
  - 关闭管道（Close）后不能再往某个管道发送值。在调用close之后，如果channel空了接收操作会返回一个零值，不会阻塞。
    
	```GO
	ch := make(chan string) // 引用类型要用make初始化
	go func() {
		ch <- "Hello!"
		close(ch)
	}()
	fmt.Println(<-ch)    // 输出字符串"Hello!"
	fmt.Println(<-ch)    // 输出零值 - 空字符串""，不会阻塞
	fmt.Println(<-ch)    // 再次打印输出空字符串"",不会阻塞
	v, ok := <-ch        // 变量v的值为空字符串""，变量ok的值为false
	fmt.Println(v, ok)
	```

	```GO
	// 一个带有range子句的for语句会依次读取发往管道的值，直到该管道关闭：
	type Sushi string
	
	func main() {
		// 译注：要想运行该示例，需要先定义类型Sushi，如type Sushi string
		var ch <-chan Sushi = Producer() // "<-chan Sushi"是数据类型。
		var i int
		for s := range ch {
			i++
			fmt.Println("Consumed", s, " No. " + strconv.Itoa(i))
		}
	}
	
	func Producer() <-chan Sushi {
		ch := make(chan Sushi)
		go func(){
			ch <- Sushi("海老握り")    // Ebi nigiri
			ch <- Sushi("鮪とろ握り") // Toro nigiri
			close(ch)
		}()
		return ch
	}
	// 输出为
	Consumed 海老握り  No. 1
	Consumed 鮪とろ握り  No. 2
	```
  - buffered channel 缓冲信道
    - wc := make(chan *Work, 10)    // 带缓冲的Work类型指针管道
  - 同步
```GO
// deadlock例子
func f1(in chan int) {
    fmt.Println(<-in)
}

func main() {
    out := make(chan int) // 默认容量为0
    out <- 2 // 此处就会阻塞
    go f1(out)
}
```

```GO
// 解决方案1
func main() {
	out := make(chan int, 1) // 容量变为1
	out <- 2
	go f1(out)
}
// 解决方案2
func main() {
	out := make(chan int)
	go f1(out)  // 先启动一个读取数据的协程
	out <- 2
}
```

规律为：  
假设有 ch :=make(chan type, value)  
value == 0 -> synchronous, unbuffered (阻塞）  
value > 0 -> asynchronous, buffered（非阻塞）取决于value元素  

#### 死锁:
  - 死锁是线程之间相互等待，其中任何一个都无法向前运行的情形。
  - Go语言对于运行时的死锁检测具备良好的支持。当没有任何goroutine能够往前执行的情形发生时，Go程序通常会提供详细的错误信息。以下就是我们的问题程序的输出：
  `fatal error: all goroutines are asleep - deadlock!`

#### 数据竞争（data race）
  - 当两个或更多的线程并发地访问同一个变量，并且其中至少一个访问是写操作时，数据竞争就发生了。
	```GO
	// 有并发安全性问题的版本
	package main
	 
	import "fmt"
	import "time"
	import "math/rand"
	import "runtime"
	 
	var total_tickets int32 = 10;
	 
	func sell_tickets(i int){
	    for{
	        if total_tickets > 0 { //如果有票就卖
	            time.Sleep( time.Duration(rand.Intn(5)) * time.Millisecond)
	            total_tickets-- //卖一张票
	            fmt.Println("id:", i, "  ticket:", total_tickets)
	        }else{
	            break
	        }
	    }
	}
	 
	func main() {
	    runtime.GOMAXPROCS(4) //我的电脑是4核处理器，所以我设置了4
	    rand.Seed(time.Now().Unix()) //生成随机种子
	 
	    for i := 0; i < 5; i++ { //并发5个goroutine来卖票
	         go sell_tickets(i)
	    }
	    //等待线程执行完
	    var input string
	    fmt.Scanln(&input)
	    fmt.Println(total_tickets, "done") //退出时打印还有多少票
	}
	
	// 通过锁来解决这个问题。关键之处在于：所有对共享数据的访问，不管读写，仅当goroutine持有锁才能操作。
	var mutex = &sync.Mutex{} //可简写成：var mutex sync.Mutex
 
	func sell_tickets(i int){
	    for total_tickets > 0 {
	        mutex.Lock()
	        if total_tickets > 0 {
	            time.Sleep( time.Duration(rand.Intn(5)) * time.Millisecond)
	            total_tickets--
	            fmt.Println(i, total_tickets)
	        }
	        mutex.Unlock()
	    }
	}
	```

#### 原子操作  TODO：

#### 检测数据竞争
#### Select语句
- select 做的就是：选择处理列出的多个通信情况中的一个。
  - 如果都阻塞了，会等待直到其中一个可以处理
  - 如果多个可以处理，随机选择一个
  - 如果没有通道操作可以处理并且写了 default 语句，它就会执行： default 永远是可运行的（这就是准备好了，可以执行）。
- 使用 select轮询的例子

	```GO
	select {
	case u:= <- ch1:
	        ...
	case v:= <- ch2:
	        ...
	        ...
	default: // no value ready to be received
	        ...
	}
	```
- 两种实际的用法：一种是阻塞但有timeout，一种是无阻塞。
  - 来看看如果给select设置上timeout的
  ```GO
	func main() {
		c1 := make(chan string)
		c2 := make(chan string)
		timeout_cnt := 0
		for {
			select {
			case msg1 := <-c1:
				fmt.Println("msg1 received", msg1)
			case msg2 := <-c2:
				fmt.Println("msg2 received", msg2)
			case <-time.After(time.Second * 30): // 会停滞30秒
				fmt.Println("Time Out")
				timeout_cnt++
			}
			if timeout_cnt > 3 {
				break
			}
		}
	}
  ```
  - 在select中加入default实现无阻塞
  ```GO
	for {
		select {
		case msg1 := <-c1:
			fmt.Println("received", msg1)
		case msg2 := <-c2:
			fmt.Println("received", msg2)
		case i = i+1://<-随意放一个不阻塞的操作在这里，意图避免阻塞。结果运行时出错： select assignment must have receive on right hand side
		default: // 依靠 default 实现无阻塞
			fmt.Println("nothing received!")
			time.Sleep(time.Second)
		}
	}
  ```

#### 并行计算
```GO
numcpu := runtime.NumCPU()
runtime.GOMAXPROCS(numcpu) // 尝试使用所有可用的CPU
```

#### 定时器 Timer
- 等待一段时间，类似于sleep
	```GO
	func main() {
	    timer := time.NewTimer(2*time.Second)
	 
	    <- timer.C
	    fmt.Println("timer expired!")
	}
	```
- 每隔一段时间触发一次
	```Go
	func main() {
	    ticker := time.NewTicker(time.Second)
	 
	    go func () {
	        for t := range ticker.C {
	            fmt.Println(t)
	        }
	    }()
	 
	    //设置一个timer，10钞后停掉ticker
	    timer := time.NewTimer(10*time.Second)
	    <- timer.C
	 
	    ticker.Stop()
	    fmt.Println("timer expired!")
	}
	```
#### 疑问 misc
sleep() 非阻塞，放弃cpu时间


#### 参考资料
- [并发,并行和协程](http://blog.xiayf.cn/2015/05/20/fundamentals-of-concurrent-programming/)
- [Go 语言简介（下）](http://coolshell.cn/articles/8489.html)
- [the-way-to-go](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/14.4.md)
- [The Go Programming Language](https://docs.ruanjiadeng.com/gopl-zh/ch8/ch8.html)
