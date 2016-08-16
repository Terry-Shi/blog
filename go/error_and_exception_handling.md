### Go 错误和异常处理

#### 利用 GO 语言中函数可以返回多个值的特性，通过返回值传递出错信息
尽可能使用返回值，而不是抛出异常来反馈错误信息
```Go
func myfunc() (retVal retValType, err error){
    // something is wrong
    return nil, errors.New("Bad Arguments - negtive!")
}
```

#### 语言内置了error接口，用户定义的异常可以实现这个接口
```Go
type error interface {
	Error() string // 自定义异常只需要实现Error()方法
}

//自定义的出错数据结构
type myError struct {
    arg  int
    errMsg string
}
//实现Error接口
func (e *myError) Error() string {
    return fmt.Sprintf("%d - %s", e.arg, e.errMsg)
}
```
* Go语言的接口实现方式对可读性是不利的！ 很难一眼看出一个struct实现了哪些接口

#### panic， defer 和 recover
- 与 java 不同，Go语言没有采用try catch finally的异常处理机制。
而用panic（类似于throw） defer（类似于finally） recover（类似于catch）这一套机制
```Go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()
 
    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()
 
    return io.Copy(dst, src)
}
```

- 多个defer采用LIFO的顺序执行
```Go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
//被延期的函数以后进先出（LIFO）的顺行执行，因此以上代码在返回时将打印4 3 2 1 0。
```
- 一个不如try catch finally方便的场景：循环中的defer
```Go
func ...
	for /* stuff */ {
	     f, err := os.Open()
	     if err != nil {
	         // do somthing with f
	     }
	     defer f.Close() // 编译器会报 resource leak 警告，因为这里f.close不会执行
	}
	// something else 
}
// 可以把defer放到匿名函数中，这样每一次循环内defer都会执行
func ...
	for /* stuff */ {
	     func() {
	         f, err := os.Open(/*stuff*/)
	         if err != nil { 
	             // do somthing with f
	         }
	         defer f.Close()
	     }() // <-- note parens making this a call
	}
	// something else 
}
```
- defer是在函数return后执行。但是如果函数先执行了panic，则后面的defer会被忽略。 TODO：
```Go
	f, err := os.Open("c")
	if err != nil {
		// do somthing with f
		panic("panic when open file")
	}
	defer f.Close() //  如果err!=nil, 这里的defer不会被执行。
```



