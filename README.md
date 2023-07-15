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

/*--- iota 特殊常量 ---*/
/*  iota 在 const关键字出现时将被重置为 0(const 内部的第一行之前)，
    const 中每新增一行常量声明将使 iota 计数一次(iota 可理解为 const 语句块中的行索引)。  */
// 常用来做枚举值
const (
    a = iota  // 0
    b         // 1
    c         // 2
)
const (
   a = iota   //0
   b          //1
   c          //2
   d = "ha"   //独立值，iota += 1
   e          //"ha"   iota += 1
   f = 100    //iota +=1
   g          //100  iota +=1
   h = iota   //7,恢复计数
   i          //8
)
const (
   i=1<<iota  //1, iota = 0
   j=3<<iota  //6, iota = 1 
   k          //12即 3<<2 iota = 2 
   l          //24即 3<<3 iota = 3 
)
```
### 运算符
* 算数运算符  
  \+ \- \* / % ++ --（同cpp）
* 关系运算符  
  == != > < >= <= （同cpp）
* 逻辑运算符  
  && || !（同cpp）
* 位运算符  
  & | ^ << >>（同cpp）
* 赋值运算符  
  = += -= \*= /= %= <<= >>= &= ^= |=（同cpp）
* 运算符优先级
![image-1](https://github.com/lizyzzz/LearnGolang/blob/main/images/1.png)
### 条件语句
（1）if 语句
```
package main
import "fmt"

func main() {
   /* go 没有三目运算符 ? : */
   /* 除了条件表达式不带括号, else if 和 else 不能换行, 其他跟 cpp 一样 */
   var a int = 10
   if a < 20 {  // 条件表达式不带括号
       fmt.Printf("a 小于 20\n" )
   }
   fmt.Printf("a 的值为 : %d\n", a)

   if a < 20 {
       fmt.Printf("a 小于 20\n" );
   } else {  /* else 不能换行 */
       fmt.Printf("a 不小于 20\n" );
   }
   // if else if else
   var age int = 23
   if age == 25 {
     fmt.Println("true")
   } else if age < 25 {
     fmt.Println("too small")
   } else {
     fmt.Println("too big")
   }
}
```
（2）switch 语句
```
/*
switch 语句执行的过程从上至下，直到找到匹配项，匹配项后面也不需要再加 break。
switch 默认情况下 case 最后自带 break 语句，匹配成功后就不会执行其他 case，如果我们需要执行后面的 case，可以使用 fallthrough 。
*/
package main

import "fmt"

func main() {
   var marks int = 90

   switch marks {
      case 90: grade = "A"
      case 80: grade = "B"
      case 50,60,70 : grade = "C"
      default: grade = "D"  
   }
   // switch 语句还可以被用于 type-switch 来判断某个 interface 变量中实际存储的变量类型。
   switch x.(type){
      case type:
         statement(s);      
      case type:
         statement(s); 
    /* 你可以定义任意个数的case */
      default: /* 可选 */
         statement(s);
   }
   // fallthrough 情况
   switch {
      case false:
         fmt.Println("1、case 条件语句为 false")
         fallthrough
      case true:
         fmt.Println("2、case 条件语句为 true")
         fallthrough
      case false:
         fmt.Println("3、case 条件语句为 false")
         fallthrough
      case true:
         fmt.Println("4、case 条件语句为 true")
      case false:
         fmt.Println("5、case 条件语句为 false")
         fallthrough
      default:
         fmt.Println("6、默认 case")
    }
   // 输出结果 switch 从第一个判断表达式为 true 的 case 开始执行，如果 case 带有 fallthrough，
   // 程序会继续执行下一条 case，且它不会去判断下一个 case 的表达式是否为 true。
   /*
      2、case 条件语句为 true
      3、case 条件语句为 false
      4、case 条件语句为 true
    */
}
```
（3）select 语句
```
package main

import "fmt"

func main() {
  // 定义两个通道
  ch1 := make(chan string)
  ch2 := make(chan string)

  // 启动两个 goroutine，分别从两个通道中获取数据
  go func() {
    for {
      ch1 <- "from 1"
    }
  }()
  go func() {
    for {
      ch2 <- "from 2"
    }
  }()

  // 使用 select 语句非阻塞地从两个通道中获取数据
  for {
    select {
    case msg1 := <-ch1:
      fmt.Println(msg1)
    case msg2 := <-ch2:
      fmt.Println(msg2)
    default:
      // 如果两个通道都没有可用的数据，则执行这里的语句
      fmt.Println("no message received")
    }
  }
}
```
### 循环语句
（1）for 循环（Go 没有 while 循环）
```
// 第一种：和 cpp 的 for 一样
for init; condition; post { }
// 第二种：和 cpp 的 while 一样
for condition { }
// 第三种：和 cpp 的 for( ; ; ) 一样
for { }
// range 用法与 cpp 种 auto 遍历用法一样
--------------------------------
// 第一种
sum := 0
for i := 0; i <= 10; i++ {
   sum += i
}
// 第二种
sum := 1
for ; sum <= 10; {
   sum += sum
}
// 这样写也可以，更像 While 语句形式
for sum <= 10{
   sum += sum
}
// 第三种
sum := 0
for {
   sum++ // 无限循环下去
}
// range 用法
strings := []string{"google", "runoob"}
for i, s := range strings {
   fmt.Println(i, s)
}
// 输出
/*
0 google
1 runoob
*/
-----------------------------
map1 := make(map[int]float32)
map1[1] = 1.0
map1[2] = 2.0
map1[3] = 3.0
map1[4] = 4.0

// 读取 key 和 value
for key, value := range map1 {
   fmt.Printf("key is: %d - value is: %f\n", key, value)
}
// 读取 key
for key := range map1 {
   fmt.Printf("key is: %d\n", key)
}
// 读取 value
for _, value := range map1 {
   fmt.Printf("value is: %f\n", value)
}

// 跳出循环的语句 与 cpp 类似
break
continue
goto(与 cpp 一样少用)
```
### 函数
```
// 函数形式
// [] 中的内容可以省略, 而且 return_types 可以有多个值
// 默认是 值传递, 也有引用传递, 引用传递就是指针类型
func function_name( [parameter list] ) [return_types] {
   函数体
}
-------------------
package main

import "fmt"

func main() {
   var a int = 100
   var b int = 200
   var ret int

   ret = max(a, b)
   fmt.Printf( "最大值是 : %d\n", ret )
}

/* 函数返回两个数的最大值 */
func max(num1, num2 int) int {
   var result int
   if (num1 > num2) {
      result = num1
   } else {
      result = num2
   }
   return result
}
--------------------
func swap(x, y string) (string, string) {
   return y, x
}

func main() {
   a, b := swap("Google", "Runoob")
   fmt.Println(a, b)
}
--------------------------------
// 也可以像 cpp 一样 定义函数指针类型, 并且作为函数参数, 实现回调
package main
import "fmt"

// 声明一个函数类型
type cb func(int) int

func main() {
    testCallBack(1, callBack)
    testCallBack(2, func(x int) int {
        fmt.Printf("我是回调，x：%d\n", x)
        return x
    })
}

func testCallBack(x int, f cb) {
    f(x)
}

func callBack(x int) int {
    fmt.Printf("我是回调，x：%d\n", x)
    return x
}

```
### 函数闭包
Go 语言支持匿名函数，可作为闭包。匿名函数是一个"内联"语句或表达式。匿名函数的优越性在于可以直接使用函数内的变量，不必申明。
```
package main

import "fmt"

func getSequence() func() int {
   i:=0
   return func() int {
      i+=1
     return i  
   }
}

func main(){
   /* nextNumber 为一个函数，函数 i 为 0 */
   nextNumber := getSequence()  

   /* 调用 nextNumber 函数，i 变量自增 1 并返回 */
   fmt.Println(nextNumber())
   fmt.Println(nextNumber())
   fmt.Println(nextNumber())
   
   /* 创建新的函数 nextNumber1，并查看结果 */
   nextNumber1 := getSequence()  
   fmt.Println(nextNumber1())
   fmt.Println(nextNumber1())
}
// 输出结果
1 2 3 1 2
```
### 函数方法
Go 语言中同时有函数和方法。一个方法就是一个包含了接受者的函数，接受者可以是命名类型或者结构体类型的一个值或者是一个指针。所有给定类型的方法属于该类型的方法集。
```
// 语法格式:
func (variable_name variable_data_type) function_name() [return_type]{
   /* 函数体*/
}

package main

import (
   "fmt"  
)

/* 定义结构体 */
type Circle struct {
  radius float64
}

func main() {
  var c1 Circle
  c1.radius = 10.00
  fmt.Println("圆的面积 = ", c1.getArea())
}

//该 method 属于 Circle 类型对象中的方法
func (c Circle) getArea() float64 {
  //c.radius 即为 Circle 类型对象中的属性
  return 3.14 * c.radius * c.radius
}
```





