# LearnGolang
Golang学习笔记
## 1. 语法基础
### 基础组成部分：  
（1）包声明；（2）引入包；（3）函数；（4）变量；（5）语句 & 表达式；（6）注释  
```
package main
/* 定义了包名。必须在源文件中非注释的第一行指明这个文件属于哪个包，
如：package main。package main表示一个可独立执行的程序，每个 Go 应用程序都包含一个名为 main 的包。*/

import "fmt" /* 告诉 Go 编译器这个程序需要使用 fmt 包, 相当于 inclue */

func main() { // 需要注意的是 { 不能单独放在一行
   /* 这是我的第一个简单的程序 */
   fmt.Println("Hello, World!")
}
```
注意：在 Go 程序中，一行代表一个语句结束。每个语句不需要像 C 家族中的其它语言一样以分号 ; 结尾，因为这些工作都将由 Go 编译器自动完成。
如果你打算将多个语句写在同一行，它们则必须使用 ; 人为区分，但在实际开发中我们并不鼓励这种做法。  
#### 格式化字符串
Go 语言中使用 fmt.Sprintf 或 fmt.Printf 格式化字符串并赋值给新串：  
* Sprintf 根据格式化参数生成格式化的字符串并返回该字符串。
* Printf 根据格式化参数生成格式化的字符串并写入标准输出。
```
package main

import (
    "fmt"
)

func main() {
   // %d 表示整型数字，%s 表示字符串
    var stockcode=123
    var enddate="2020-12-31"
    var url="Code=%d&endDate=%s"
    var target_url=fmt.Sprintf(url,stockcode,enddate)
    fmt.Printf(url,stockcode,enddate)
    fmt.Println(target_url)
}
// 输出结果
Code=123&endDate=2020-12-31
```
### 编译与执行：  
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
关于包有以下几点值得注意：  
（1）文件名与包名没有直接关系，不一定要将文件名与包名定成同一个。  
（2）文件夹名与包名没有直接关系，并非需要一致。  
（3）同一个文件夹下的文件只能有一个包名，否则编译报错。  
```
---- 文件结构 -------
Test
--helloworld.go

myMath
--myMath1.go
--myMath2.go
------- 测试代码 -------
// helloworld.go
package main

import (
"fmt"
"./myMath"
)

func main(){
    fmt.Println("Hello World!")
    fmt.Println(mathClass.Add(1,1))
    fmt.Println(mathClass.Sub(1,1))
}
-----------------------
// myMath1.go
package mathClass
func Add(x,y int) int {
    return x + y
}
// myMath2.go
package mathClass
func Sub(x,y int) int {
    return x - y
}
```
### 数据类型
(1) 布尔型 bool; (2) 数字类型 int, uint8, uint16, ..., int64等, 浮点型 float32 float64 complex64 complex128;  
(3) 字符串类型 string 占16字节，前8字节是一个指针，后8字节是一个整数，标识字节长度。该字符串不以'\0'结尾。  
(4) 派生类型：(a) 指针 (b) 数组 (c) 结构化类型 (d) Channel类型 (e) 函数类型 (f) 切片类型 (g) 接口类型 (h) Map类型。
### 变量
声明变量用 var 关键字  
```
var identifier type
var identifier1, identifier2 type  // 声明多个变量
package main
import "fmt"
func main() {
    // 第一种方式
    var a string // 声明但没有初始化，变量默认为 "零" 值
    a = "lizy";   // 初始化

    var a string = "Runoob" // 声明并初始化
    fmt.Println(a)

    // 第二种方式（根据值推断变量类型）
    var b, c = 1, "lizy" // 多变量同样的用法
    fmt.Println(b)

    // 第三种方式
    c := 1 // 声明并初始化变量c
}
```
### 常量
```
// 定义格式： const identifier [type] = value // [type] 可以省略
const LENGTH int = 10
const WIDTH int = 5  
var area int = LENGTH * WIDTH
const a, b, c = 1, false, "str" //多重赋值
```





