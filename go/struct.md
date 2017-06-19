### Go struct 结构体

*  Go 语言中没有类的概念

#### 1 如何定义
```go
type Person struct {
    name string
    age  int
    email string
}

func main() {
    //初始化
    person := Person{"Tom", 30, "tom@gmail.com"}
    person = Person{name:"Tom", age: 30, email:"tom@gmail.com"}

    fmt.Println(person) //输出 {Tom 30 tom@gmail.com}

    pPerson := &person

    fmt.Println(pPerson) //输出 &{Tom 30 tom@gmail.com}

    pPerson.age = 40  // 指针可以直接用"."来取数据和调用方法
    person.name = "Jerry"
    fmt.Println(person) //输出 {Jerry 40 tom@gmail.com}
}
```


```go
type rect struct {
    width, height int
}

// (r *rect) 表示这个函数是属于结构体rect的
func (r *rect) area() int { //求面积
    return r.width * r.height
}

func (r *rect) perimeter() int{ //求周长
    return 2*(r.width + r.height)
}

func main() {
    r := rect{width: 10, height: 15}

    fmt.Println("面积: ", r.area())
    fmt.Println("周长: ", r.perimeter())

    rp := &r
    fmt.Println("面积: ", rp.area())
    fmt.Println("周长: ", rp.perimeter())
}
```

#### 2 初始化
初始化一个结构体实例（一个结构体字面量：struct-literal）的更简短和惯用的方式如下：
&struct1{a, b, c} 是一种简写，底层仍然会调用 new ()，这里值的顺序必须按照字段顺序来写。在下面的例子中能看到可以通过在值的前面放上字段名来初始化字段的方式。表达式 new(Type) 和 &Type{} 是等价的。
    ms := &struct1{10, 15.5, "Chris"}
    // 此时ms的类型是 *struct1
或者：
    var ms struct1
    ms = struct1{10, 15.5, "Chris"}

#### 3 函数首字母大写 表示可以被其他pkg调用
Go语言中没有public, protected, private的关键字，所以，如果你想让一个方法可以被别的包访问的话，你需要把这个方法的第一个字母大写。这是一种约定。

#### 4 结构体工厂
Go 语言不支持面向对象编程语言中的构造方法，但是可以很容易的在 Go 中实现 “构造子工厂”方法。为了方便通常会为类型定义一个工厂，按惯例，工厂的名字以 new 或 New 开头。假设定义了如下的 File 结构体类型：
https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/10.2.md

#### 匿名字段和内嵌结构体
结构体可以包含一个或多个 匿名（或内嵌）字段，即这些字段没有显式的名字，只有字段的类型是必须的，此时类型就是字段的名字。匿名字段本身可以是一个结构体类型，即 结构体可以包含内嵌结构体。在一个结构体中对于每一种数据类型只能有一个匿名字段。
Go 语言中的继承是通过内嵌或组合来实现的.
https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/10.5.md

#### 方法 method
在方法名之前，func 关键字之后的括号中指定 receiver
func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }

类型和方法之间的关联由接收者来建立。

如果方法不需要使用 recv 的值，可以用 _ 替换它，比如：
func (_ receiver_type) methodName(parameter_list) (return_value_list) { ... }

#### 指针or值作为接受者
指针作为接受者，可以改变接受者的值
```go
type B struct {
    thing int
}

func (b *B) change() { b.thing = 1 }

func (b B) write() string { return fmt.Sprint(b) }

func (b B) addOne() string {
    b.thing = b.thing + 1
    return fmt.Sprint(b)
}

func main() {
    var b1 B // b1是值
    fmt.Println(b1.write()) // default value = 0
    b1.change()  // change to 1
    fmt.Println(b1.write()) // print 1

    b2 := new(B) // b2是指针
    b2.change() // 1
    fmt.Println(b2.write()) // 1
    fmt.Println(b2.addOne()) // 2
    fmt.Println(b2.write()) // 1
}
```


#### 继承（内嵌struct interface）



