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
  - 阻塞式读写。写操作如果channel满了就会阻塞，读操作如果channel空了就会阻塞。
  - 关闭管道（Close）后不能再往某个管道发送值。在调用close之后，并且在之前发送的值都被接收后，接收操作会返回一个零值，不会阻塞。
buffered channel  缓冲信道
  - wc := make(chan *Work, 10)    // 带缓冲的Work类型指针管道
同步
- 死锁:死锁是线程之间相互等待，其中任何一个都无法向前运行的情形。
- 数据竞争（data race）- 当两个或更多的线程并发地访问同一个变量，并且其中至少一个访问是写操作时，数据竞争就发生了。
- 互斥锁: sync.Mutex 要想这类加锁起效的话，关键之处在于：所有对共享数据的访问，不管读写，仅当goroutine持有锁才能操作。
- 检测数据竞争
