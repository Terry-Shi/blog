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
  - main函数的协程结束，其他协程也立刻结束

#### channel 管道或信道 
  - 用于两个goroutine之间通过传递一个指定类型的值来同步运行和通讯。
  - 操作符<-用于指定管道的方向，发送或接收。如果未指定方向，则为双向管道。  
	```GO
	// 以下是三种不同类型的定义，
	chan Sushi        // 可用来发送和接收Sushi类型的值  
	chan<- float64    // 仅可用来接受float64类型的值  
	<-chan int        // 仅可用来发送int类型的值   （不能接受，只能发送有什么意义?）
	// TODO： 不同管道类型之间赋值兼容性 （错误的类型赋值会在运行时抛出异常）
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
  - select是阻塞-是什么含义?
  - 使用 select 切换协程
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
  
#### 并行计算
```GO
    numcpu := runtime.NumCPU()
    runtime.GOMAXPROCS(numcpu) // 尝试使用所有可用的CPU
```

#### 参考资料
- [并发,并行和协程](http://blog.xiayf.cn/2015/05/20/fundamentals-of-concurrent-programming/)
- [Go 语言简介（下）](http://coolshell.cn/articles/8489.html)
- [the-way-to-go](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/14.4.md)
