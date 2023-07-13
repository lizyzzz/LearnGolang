# LearnGolang
Golang学习笔记
## 1. 语法基础
基础组成部分：  
（1）包声明；（2）引入包；（3）函数；（4）变量；（5）语句 & 表达式；（6）注释  
```
package main
/* 定义了包名。必须在源文件中非注释的第一行指明这个文件属于哪个包，
如：package main。package main表示一个可独立执行的程序，每个 Go 应用程序都包含一个名为 main 的包。*/

import "fmt" /* 告诉 Go 编译器这个程序需要使用 fmt 包, 相当于 inclue */

func main() {
   /* 这是我的第一个简单的程序 */
   fmt.Println("Hello, World!")
}
```
编译与执行：  
```
$ go run hello.go
Hello, World!
或
$ go build hello.go 
$ ls
hello    hello.go
$ ./hello 
Hello, World!
```

