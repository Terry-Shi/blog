### Go 语言函数
#### Go 里面有三种类型的函数：
- 普通的带有名字的函数
- 匿名函数或者lambda函数（参考 第 6.8 节）
- 方法（Methods，参考 第 10.6 节）

#### 函数被调用的基本格式如下：  
pack1.Function(arg1, arg2, …, argn)

#### 在 Go 里面函数重载是不被允许的。这将导致一个编译错误：
funcName redeclared in this book, previous declaration at line no X

#### 目前 Go 没有泛型（generic）的概念

#### 按值传递（call by value） 按引用传递（call by reference）
- Go 默认使用按值传递来传递参数，也就是传递参数的副本。函数接收参数副本之后，在使用变量的过程中可能对副本的值进行更改，但不会影响到原来的变量，比如 Function(arg1)。  
- 如果你希望函数可以直接修改参数的值，而不是对参数的副本进行操作，你需要将参数的地址（变量名前面添加&符号，比如 &variable）传递给函数，这就是按引用传递，比如 Function(&arg1)，此时传递给函数的是一个指针。  
- 在函数调用时，像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显示的指出指针）。

#### 变长参数 （数量不确定的参数） Java5开始也有同样功能
func Greeting(prefix string, who ...string) // 此时 who是字符串数组
Greeting("hello:", "Joe", "Anna", "Eileen")


*argument与parameter的区别：形参，实参
*function与method的区别：函数，方法

#### REF
https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/06.2.md
