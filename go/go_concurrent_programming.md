### Go 并发编程

#### 参考资料 [并发,并行和协程] (http://blog.xiayf.cn/2015/05/20/fundamentals-of-concurrent-programming/)

- goroutine 协程 - 比线程更加轻量级
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
- channel 管道或信道 - 用于两个goroutine之间通过传递一个指定类型的值来同步运行和通讯。
  - 操作符<-用于指定管道的方向，发送或接收。如果未指定方向，则为双向管道。  
	```GO
	// 以下是三种不同类型的定义，
	chan Sushi        // 可用来发送和接收Sushi类型的值  
	chan<- float64    // 仅可用来发送float64类型的值   ？ 不能接受，只能发送有什么意义
	<-chan int        // 仅可用来接收int类型的值  
	// TODO： 不同管道类型之间赋值兼容性 （错误的类型赋值会在运行时抛出异常）
	var receiveChan <-chan int
	var sendChan chan<- int
	var twoWayChan chan int 
	
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
    - wc := make(chan *Work, 10)    // 带缓冲的Work类型指针管道（容量更大）
- 同步
- 死锁:死锁是线程之间相互等待，其中任何一个都无法向前运行的情形。
- 数据竞争（data race）- 当两个或更多的线程并发地访问同一个变量，并且其中至少一个访问是写操作时，数据竞争就发生了。
- 互斥锁: sync.Mutex 要想这类加锁起效的话，关键之处在于：所有对共享数据的访问，不管读写，仅当goroutine持有锁才能操作。
- 检测数据竞争
