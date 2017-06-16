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
