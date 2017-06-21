# godep 包依赖的管理工具

1. go1.6 开始内置了vendoring功能。
参考 https://stackoverflow.com/questions/37237036/how-should-i-use-vendor-in-go-1-6

假设 import github.com/zenazn/goji
go build or go run的搜寻顺序：1. 工程目录下的vendor子目录  2. $GOPATH 3. $GOROOT
./vendor/github.com/zenazn/goji
$GOPATH/src/github.com/zenazn/goji
$GOROOT/src/github.com/zenazn/goji

With that said, `go get` will continue to install into you $GOPATH/src; and, go install will install into $GOPATH/bin for binaries or $GOPATH/pkg for package caching.


2. dependency management tool
godep 参考 http://www.cnblogs.com/me115/p/5528463.html
编译／运行
项目用godep管理后，要编译和运行项目的时候再用go run和go build显然就不行了，因为go命令是直接到GOPATH目录下去找第三方库。
而使用godep下载的依赖库放到Godeps/workspace目录下的；
`godep go build XXX`
godep中的go命令，就是将原先的go命令加了一层壳，执行godep go的时候，会将当前项目的workspace目录加入GOPATH变量中；


3. GOPATH 环境变量
$GOPATH 目录约定有三个子目录：
src 存放源代码（比如：.go .c .h .s等）
pkg 编译后生成的文件（比如：.a）
bin 编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中）


4. how to setup GOPATH in IntelliJ IDEA

5. 命令行方式 run go program under workspace
官方说明 https://golang.org/doc/code.html#Workspaces
https://saintcoder.wordpress.com/2017/01/28/how-to-setup-your-first-go-langs-workspace-and-run-your-1st-hello-world-in-your-linux-box/

6. 自定义包的可见性／可访问性
https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/09.5.md
```go
//　工程目录根目录下有 main.go
package main

import (
    "./test_struct" // cannot resolve file 'test_struct'
)

func main() {

}

// 同一目录下有 mystruct.go 其中 package test_struct

```
一般规则 import pkgNameA 表示 有源文件在 pkgNameA 目录下且第一行为package pkgNameA
修正这个错误的一个做法是：
修改 main.go
import (
    "./test_struct" // 使用相对目录
)
创建 test_struct目录，把 mystruct.go 启动到这个目录下。


REF:
