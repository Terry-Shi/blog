# go 语言的 package 管理

1. 目录结构／GOPATH
参考：
官方说明 https://golang.org/doc/code.html#Workspaces
https://openhome.cc/Gossip/Go/HelloWorld.html
https://openhome.cc/Gossip/Go/Package.html

GO 语言的设计是所有工程共用一个workspace，workspace的具体位置由 $GOPATH 决定。
$GOPATH 约定有三个子目录：
src 存放源代码（比如：.go .c .h .s等）
pkg 编译后生成的文件（比如：.a）
bin 编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中）
```
bin/
    hello                          # command executable
    outyet                         # command executable
pkg/
    linux_amd64/
        github.com/golang/example/
            stringutil.a           # package object
src/
    github.com/golang/example/
        .git/                      # Git repository metadata
	hello/
	    hello.go               # command source
	outyet/
	    main.go                # command source
	    main_test.go           # test source
	stringutil/
	    reverse.go             # package source
	    reverse_test.go        # test source
    golang.org/x/image/
        .git/                      # Git repository metadata
	bmp/
	    reader.go              # package source
	    writer.go              # package source
    ... (many more repositories and packages omitted) ...
```

* import {path of package；NOT package name} 注意import语句本质的含义是去后面的path搜寻源文件。
* go 文件第一行 package {packge name}
* 本地目录与远程目录对应关系：以 github.com/golang/example/ 为例
   远程含义：golang 是 github 用户名； example 是 repo 名
   本地含义：example 是 package name
* 一般原则：源文件所处的目录名需要符合 package 的名字。main package可以除外。
假设
export GOPATH=~\workspace\go
~go
 |~src/
 | |~hello/
 | | `-hello.go <- hello.go 的第一行是 package hello
 `-main.go
go run main.go - 产生名字为 main 的可执行文件。
也可以建立一個 bin 目錄，然後執行 go build -o bin/main main.go，這樣產生出來的可執行檔，就會被放在 bin 底下。
如果想將原始碼全部放在 src 底下管理，那麼就將 main.go 放到 src/main 底下，然後執行 go build -o bin/main src/main/main.go。

* 一个 package 的目录下可以包含多个go文件，例如 go install goexample 会在 pkg 目錄的 $GOOS_$GOARCH 目錄中產生 goexample.a 檔案
* 也可以在 package 目录前增加父目录，可以建立一個 src/cc/openhome 目錄，然後將方才的 hello.go 與 hi.go 移至該目錄之中，接著執行 go install cc/openhome/goexample，那麼，在 pkg 目錄的 $GOOS_$GOARCH 目錄中，會產生對應的 cc/openhome 目錄，其中放置著 goexample.a 檔案
远程 package ：例如，你可以建立 src/github.com/JustinSDK 目錄，然後將方才的 goexample 目錄移到 src/github.com/JustinSDK 當中
```go
ackage main

import "github.com/JustinSDK/goexample"

func main() {
        goexample.Hi()
        goexample.Hello()
}
```
发布到 github 后，其他项目可以依赖它
go get github.com/JustinSDK/goexample
```
|~bin/
| |-main*
|~pkg/
| `~linux_arm/
|   |~github.com/
|   | |~JustinSDK/
|   | | `-goexample.a
`~src/
  |~github.com/
  | |~JustinSDK/
  | | `~goexample/
  | |   |-hello.go
  | |   |-hi.go
  | |   `-README.md
  `~main/
    `-main.go
```



2. go 1.6 开始内置了vendoring功能。
参考 https://stackoverflow.com/questions/37237036/how-should-i-use-vendor-in-go-1-6

假设 import github.com/zenazn/goji
go build or go run的搜寻顺序：1. 工程目录下的vendor子目录  2. $GOPATH 3. $GOROOT
./vendor/github.com/zenazn/goji
$GOPATH/src/github.com/zenazn/goji
$GOROOT/src/github.com/zenazn/goji

`go get` will continue to install into you $GOPATH/src; and, go install will install into $GOPATH/bin for binaries or $GOPATH/pkg for package caching.


3. dependency management tool
godep 参考 http://www.cnblogs.com/me115/p/5528463.html
编译／运行
项目用godep管理后，要编译和运行项目的时候再用go run和go build显然就不行了，因为go命令是直接到GOPATH目录下去找第三方库。
而使用godep下载的依赖库放到Godeps/workspace目录下的；
`godep go build XXX`
godep中的go命令，就是将原先的go命令加了一层壳，执行godep go的时候，会将当前项目的workspace目录加入GOPATH变量中；




4. 自定义包的可见性／可访问性
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


### 目录组织结构／package 依赖管理等等方面的最佳实践
http://peter.bourgon.org/go-best-practices-2016/

