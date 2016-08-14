### go语言文件操作
主要涉及os和io两个package

#### 创建文件
```Go
//返回File的内存地址,错误信息,通过os库调用
func Create(name string) (file *File, err Error)
//返回文件的内存地址,通过os库调用
func NewFile(fd int, name string) *File
参数 fd 表示file descriptor
```
#### 打开文件
```Go
//返回File的内存地址,错误信息,通过os库调用
func Open(name string)(file *File, err Error)
//返回File的内存地址,错误信息,通过os库调用
func OpenFile(name string,flag int,perm unit32)(file *File, err Error)
```
#### 删除文件
```Go
//传入文件的路径来删除文件,返回错误个数
func Remove(name string) Error
```
#### 读文件
```Go
//读取一个slice,返回读的个数,错误信息,通过File的内存地址调用
func (file *File) Read(b []byte)(n int, err Error)
//从slice的某个位置开始读取,返回读到的个数,错误信息,通过File的内存地址调用
func (file *File) ReadAt(b []byte,off int64)(n int,err Error)

// 一个完整例子
import (
    "fmt"
    "os"
)

func main() {
  userFile := "d:/test.txt" //文件路径
  fin,err := os.Open(userFile) //打开文件,返回File的内存地址
  defer fin.Close() //延迟关闭资源
  if err != nil{
    fmt.Println(userFile,err)
    return
  }
  buf := make([]byte,1024)//创建一个初始容量为1024的slice,作为缓冲容器
  for{
    //循环读取文件数据到缓冲容器中,返回读取到的个数
    n,_ := fin.Read(buf)

    if 0==n{
        break //如果读到个数为0,则读取完毕,跳出循环
    }
    //从缓冲slice中写出数据,从slice下标0到n,通过os.Stdout写出到控制台
    os.Stdout.Write(buf[:n])
  }
}
```

#### 写文件
```Go
//写入一个slice,返回写的个数,错误信息,通过File的内存地址调用
func (file *File)Write(b []byte)(n int,err Error)
//从slice的某个位置开始写入,返回写的个数,错误信息,通过File的内存地址调用
func (file *File)WriteAt(b []byte,off int64)(n int,err Error)
//写入一个字符串,返回写的个数,错误信息,通过File的内存地址调用
func (file *File) WriteString(s string)(ret int,err Error)

// 一个完整例子
import (
    "fmt"
    "os"
)

func main() {
  userFile := "d:/test.txt" //文件路径
  fout,err := os.Create(userFile) //根据路径创建File的内存地址
  defer fout.Close() //延迟关闭资源
  if err != nil{
    fmt.Println(userFile,err)
    return
  }
  //循环写入数据到文件
  for i:=0;i<10;i++{
    fout.WriteString("Hello world!\r\n") //写入字符串
    fout.Write([]byte("abcd!\r\n"))//强转成byte slice后再写入
  }
}
```
#### 关闭文件
```Go
func (f *File) Close() error

```

#### 使用bufio库
```Go
import (
    "bufio"
    "io"
    "os"
)

func main() {
    fi, err := os.Open("input.txt")//打开输入*File
    if err != nil { panic(err) }
    defer fi.Close()
    r := bufio.NewReader(fi)//创建一个读取缓冲流

    fo, err := os.Create("output.txt")//创建输出*File
    if err != nil { panic(err) }
    defer fo.Close()
    w := bufio.NewWriter(fo)//创建输出缓冲流

    buf := make([]byte, 1024)
    for {
        n, err := r.Read(buf)
        if err != nil && err != io.EOF { panic(err) }
        if n == 0 { break }

        if n2, err := w.Write(buf[:n]); err != nil {
            panic(err)
        } else if n2 != n {
            panic("error in writing")
        }
    }

    if err = w.Flush(); err != nil { panic(err) }
}
```
#### 读取 gzip 文件
```Go
//打开文件，并进行相关处理
file, err := os.Open(fileFullPathName)
if err != nil {
	fmt.Printf("%v\n", err)
	return
}
defer file.Close()

var line string = ""
if strings.HasSuffix(fileFullPathName, ".gz") {
	//将文件作为一个io.Reader对象进行buffered I/O操作
	reader, err := gzip.NewReader(file) //  NewReader的输入参数是io.Reader接口 这个接口只有一个函数Read() 而File结构体实现了这个方法。所以自然的实现了io.Reader接口。
	br := bufio.NewReader(reader)
	for {
		line, err = br.ReadString('\n')
		if err != nil {
			break
		} else {
			fmt.Println(line)
        ｝
    ｝
｝
```

#### 使用ioutil库
```Go
func main() {
    b, err := ioutil.ReadFile("input.txt")//读文件
    if err != nil { panic(err) }

    err = ioutil.WriteFile("output.txt", b, 0644)//写文件
    if err != nil { panic(err) }
}
```

#### 遍历文件夹
遍历目录下文件（不递归）

递归遍历所有子目录＋文件


http://blog.leanote.com/post/duanhq/golang-context
#### XML文件的处理

#### JSON文件的处理


#### 参考资料
- Golang-文件操作 http://studygolang.com/articles/6563