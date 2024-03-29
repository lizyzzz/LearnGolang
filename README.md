# LearnGolang
Golang学习笔记
## 1. 语法基础
### 基础组成部分：  
（1）包声明；（2）引入包；（3）函数；（4）变量；（5）语句 & 表达式；（6）注释  
```Go
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
```Go
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
```Shell
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
```Go
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
```Go
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
![image-1](./images/1.png)
### 条件语句
（1）if 语句
```GO
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
```Go
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
```Go
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
```Go
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
```Go
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
```Go
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
```Go
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
### 数组
```Go
// 声明数组
// var arrayName [size]dataType
var balance [10]float32 // 会默认初始化为 0.0
// 数组初始化
var numbers [5]int // 默认初始化
var numbers = [5]int{1, 2, 3, 4, 5} // 初始化列表初始化
numbers := [5]int{1, 2, 3, 4, 5}
// 如果数组长度不确定，可以使用 ... 代替数组的长度，编译器会根据元素个数自行推断数组的长度
var balance = [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
// 如果设置了数组的长度，我们还可以通过指定下标来初始化元素：
//  将索引为 1 和 3 的元素初始化
balance := [5]float32{1:2.0,3:7.0}
// 如果忽略 [] 中的数字不设置数组大小，Go 语言会根据元素的个数来设置数组的大小
-------------------------------------
// 多维数组
// var variable_name [SIZE1][SIZE2]...[SIZEN] variable_type
package main

import "fmt"

func main() {
    // Step 1: 创建数组
    values := [][]int{}  // [] 不加size的是切片类型, 也叫动态数组, 与 cpp 的 vector 类似 

    // Step 2: 使用 append() 函数向空的二维数组添加两行一维数组
    row1 := []int{1, 2, 3}
    row2 := []int{4, 5, 6}
    values = append(values, row1)
    values = append(values, row2)

    // Step 3: 显示两行数据
    fmt.Println("Row 1")
    fmt.Println(values[0])
    fmt.Println("Row 2")
    fmt.Println(values[1])

    // Step 4: 访问第一个元素
    fmt.Println("第一个元素为：")
    fmt.Println(values[0][0])
}
// 输出结果
Row 1
[1 2 3]
Row 2
[4 5 6]
第一个元素为：
1
// 初始化二维数组
a := [3][4]int{  
 {0, 1, 2, 3} ,   /*  第一行索引为 0 */
 {4, 5, 6, 7} ,   /*  第二行索引为 1 */
 {8, 9, 10, 11},   /* 第三行索引为 2 */ // 这里必须要有逗号
}
// 多维数组的各个维度元素数量可以不一样
// 创建空的二维数组
animals := [][]string{}

// 创建三一维数组，各数组长度不同
row1 := []string{"fish", "shark", "eel"}
row2 := []string{"bird"}
row3 := []string{"lizard", "salamander"}

// 使用 append() 函数将一维数组添加到二维数组中
animals = append(animals, row1)
animals = append(animals, row2)
animals = append(animals, row3)

// 循环输出
for i := range animals {
  fmt.Printf("Row: %v\n", i)
  fmt.Println(animals[i])
}
// 输出结果
Row: 0
[fish shark eel]
Row: 1
[bird]
Row: 2
[lizard salamander]

// 数组可以作为函数参数, 同 cpp 一样, 类似传指针, 需要自己另外再传一个长度
func getAverage(arr []int, size int) float32 {
  // 函数体
}
```
### 指针
```Go
// 语法形式
// var var_name *var-type
// 空指针为 nil
/* 也支持:
   指针的指针
   指针数组
   等
*/
```
### 结构体
```Go
// 语法形式
type struct_variable_type struct {
   member definition
   member definition
   ...
   member definition
}
// 定义了结构体就能用于变量的声明
variable_name := structure_variable_type {value1, value2...valuen}
或
variable_name := structure_variable_type { key1: value1, key2: value2..., keyn: valuen}
// 如果要访问结构体成员，需要使用点号 "." 操作符
package main

import "fmt"

type Books struct {
   title string
   author string
   subject string
   book_id int
}

func main() {
   var Book1 Books        /* 声明 Book1 为 Books 类型 */
   var Book2 Books        /* 声明 Book2 为 Books 类型 */

   /* book 1 描述 */
   Book1.title = "Go 语言"
   Book1.author = "www.runoob.com"
   Book1.subject = "Go 语言教程"
   Book1.book_id = 6495407
   /* 打印 Book1 信息 */
   fmt.Printf( "Book 1 title : %s\n", Book1.title)
   fmt.Printf( "Book 1 author : %s\n", Book1.author)
   fmt.Printf( "Book 1 subject : %s\n", Book1.subject)
   fmt.Printf( "Book 1 book_id : %d\n", Book1.book_id)
}
------------------------------
// 结构体指针
var struct_pointer *struct_variable_type
// 使用结构体指针访问结构体成员，使用 "." 操作符(与 cpp 的 -> 不同)
```
### 切片
类似 cpp 中的 vector 数组
```Go
// 语法形式
// var identifier []type // 不用说明长度
/*
   可以使用 make() 函数来创建切片
   var slice1 []type = make([]type, len)
   slice1 := make([]type, len)
   make([]T, length, capacity)  // 指定容量
// 切片初始化
s :=[]int {1,2,3} // 直接初始化
s := arr[:]       // 以数组 arr 初始化
s := arr[startIndex:endIndex]  // s 以 arr 中从下标 startIndex 到 endIndex-1 下的元素
s := arr[startIndex:]          // 默认 endIndex 时将表示一直到arr的最后一个元素
s := arr[:endIndex]            // 默认 startIndex 时将表示从 arr 的第一个元素开始
s :=make([]int,len,cap)        // 通过内置函数 make() 初始化切片s

// len() 和 cap() 函数
len(x) 和 cap(x) 返回 x 的长度和容量
// 一个切片在未初始化之前默认为 nil，长度为 0
var numbers []int
if(numbers == nil){
   // true
}
// append() 和 copy() 函数
 // append() 相当于 push_back() 方法
var numbers []int
numbers = append(numbers, 0)
numbers = append(numbers, 1)
numbers = append(numbers, 2,3,4)
// copy() 函数, 需要保证 copy() 的结果数组有足够的长度
copy(numbers1, numbers[1:])  // 把 number 的[1,end]拷贝给 numbers1
*/
```
### 范围 range
range 关键字用于 for 循环中迭代数组(array)、切片(slice)、通道(channel)或集合(map)的元素。在数组和切片中它返回元素的索引和索引对应的值，在集合中返回 key-value 对
```Go
// 语法格式
for key, value := range oldMap {
    newMap[key] = value
}
// 只读取 key
for key := range oldMap
for key, _ := range oldMap
// 只读取 value
for _, value := range oldMap
---------------------------------
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
for i, v := range pow {
   fmt.Printf("2**%d = %d\n", i, v)
}
kvs := map[string]string{"a": "apple", "b": "banana"}
for k, v := range kvs {
   fmt.Printf("%s -> %s\n", k, v)
}
```
### map 类型
map 是一种无序的键值对的集合。与 cpp 的 map 或 unordered_map 相似
```Go
// 创建语法
map_variable := make(map[KeyType]ValueType, initialCapacity)
// 容量会动态扩展
// 使用字面量创建 Map
m := map[string]int{
    "apple": 1,
    "banana": 2,
    "orange": 3,
}
v1 := m["apple"]     // "查"
v2, ok := m["pear"]  // 如果键不存在，ok 的值为 false，v2 的值为该类型的零值
m["apple"] = 5       // "增" "改"
delete(m, "banana")  // "删"
len := len(m)        // 获取 Map 的长度
```
### 接口类型 (多态 的类似实现)
* Go 语言提供了另外一种数据类型即接口，它把所有的具有共性的方法定义在一起，任何其他类型只要实现了这些方法就是实现了这个接口。  
* 接口可以让我们将不同的类型绑定到一组公共的方法上，从而实现多态和灵活的设计。  
* Go 语言中的接口是隐式实现的，也就是说，如果一个类型实现了一个接口定义的所有方法，那么它就自动地实现了该接口。因此，我们可以通过将接口作为参数来实现对不同类型的调用，从而实现多态。
```Go
/* 定义接口 */
type interface_name interface {
   method_name1 [return_type]
   method_name2 [return_type]
   method_name3 [return_type]
   ...
   method_namen [return_type]
}
/* 定义结构体 */
type struct_name struct {
   /* variables */
}

/* 实现接口方法 */
func (struct_name_variable struct_name) method_name1() [return_type] {
   /* 方法实现 */
}
...
func (struct_name_variable struct_name) method_namen() [return_type] {
   /* 方法实现*/
}
----- 例子 ---------------------
package main

import "fmt"

type Shape interface {
    area() float64
}

type Rectangle struct {
    width  float64
    height float64
}

func (r Rectangle) area() float64 {
    return r.width * r.height
}

type Circle struct {
    radius float64
}

func (c Circle) area() float64 {
    return 3.14 * c.radius * c.radius
}

func main() {
    var s Shape

    s = Rectangle{width: 10, height: 5}
    fmt.Printf("矩形面积: %f\n", s.area())

    s = Circle{radius: 3}
    fmt.Printf("圆形面积: %f\n", s.area())
}
```
### 类型转换
```Go
// 基本格式
type_name(expression)
var a int = 10
var b float64 = float64(a)
// 字符串 / 数字 转换
package main

import (
    "fmt"
    "strconv" // 包含头文件
)

func main() {
    // 字符串 转 数字
    str := "123"
    num, err := strconv.Atoi(str)  // err 参数为错误
    if err != nil {
        fmt.Println("转换错误:", err)
    } else {
        fmt.Printf("字符串 '%s' 转换为整数为：%d\n", str, num)
    }
    // 数字 转 字符串
    num := 123
    str := strconv.Itoa(num)
    fmt.Printf("整数 %d  转换为字符串为：'%s'\n", num, str)
}
------------------------------------
// 接口类型转换
接口类型转换有两种情况：类型断言和类型转换。
类型断言用于将接口类型转换为指定类型，其语法为：
value.(type) 
或者 
value.(T)  // 其中 value 是接口类型的变量，type 或 T 是要转换成的类型。
T(value)   // 接口类型之间也可以互相转换
----------------------------------
package main
import "fmt"

func main() {
   var i interface{} = "Hello, World"
   str, ok := i.(string)
   if ok {
     fmt.Printf("'%s' is a string\n", str)
   } else {
     fmt.Println("conversion failed")
   }
}
----------------------------------
package main

import "fmt"

type Writer interface {
    Write([]byte) (int, error)
}

type StringWriter struct {
    str string
}

func (sw *StringWriter) Write(data []byte) (int, error) {
    sw.str += string(data)
    return len(data), nil
}

func main() {
    var w Writer = &StringWriter{}
    sw := w.(*StringWriter) // 接口类型转换为 *StringWriter (相当于基类转换为子类)
    sw.str = "Hello, World"
    fmt.Println(sw.str)
}
```
### 错误处理
Go 语言通过内置的错误接口提供了非常简单的错误处理机制。
```Go
// 定义 error 接口类型
type error interface {
    Error() string
}
// 我们可以在编码中通过实现 error 接口类型来生成错误信息。
// 函数通常在最后的返回值中返回错误信息。
// 使用 errors.New 可返回一个错误信息
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // 实现
}
// 调用代码
result, err:= Sqrt(-1)
if err != nil {
   fmt.Println(err)
}
```
### 并发
Go 语言支持并发，我们只需要通过 go 关键字来开启 goroutine 即可。
goroutine 是轻量级线程（协程），goroutine 的调度是由 Golang 运行时进行管理的。
```Go
// 语法形式
go 函数名( 参数列表 )
go f(x, y, z) // 开启一个线程
-------------------------
package main

import (
   "fmt"
   "time"
)

func say(s string) {
   for i := 0; i < 5; i++ {
      time.Sleep(100 * time.Millisecond)
      fmt.Println(s)
   }
}

func main() {
   go say("world")
   say("hello")
}
// 输出(顺序不定)
world
hello
hello
world
world
hello
hello
world
world
hello
```
### 通道
通道（channel）是用来传递数据的一个数据结构。  
通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。 操作符 <- 用于指定通道的方向，发送或接收。如果未指定方向，则为双向通道。
```Go
ch <- v    // 把 v 发送到通道 ch
v := <-ch  // 从 ch 接收数据
           // 并把值赋给 v

// 声明一个通道很简单，我们使用chan关键字即可，通道在使用前必须先创建：
ch := make(chan int)
/*
注意：默认情况下，通道是不带缓冲区的。发送端发送数据，同时必须有接收端相应的接收数据。
// 通道可以设置缓冲区，通过 make 的第二个参数指定缓冲区大小：
ch := make(chan int, 100)
不过由于缓冲区的大小是有限的，所以还是必须有接收端来接收数据的，否则缓冲区一满，数据发送端就无法再发送数据了。
如果通道不带缓冲，发送方会阻塞直到接收方从通道中接收了值。
如果通道带缓冲，发送方则会阻塞直到发送的值被拷贝到缓冲区内；
如果缓冲区已满，则意味着需要等待直到某个接收方获取到一个值。接收方在有值可以接收之前会一直阻塞。
*/
package main

import "fmt"

func sum(s []int, c chan int) {
   sum := 0
   for _, v := range s {
      sum += v
   }
   c <- sum // 把 sum 发送到通道 c
}

func main() {
   s := []int{7, 2, 8, -9, 4, 0}
   
   c := make(chan int)
   go sum(s[:len(s)/2], c)
   go sum(s[len(s)/2:], c)
   x, y := <-c, <-c // 从通道 c 中接收
   
   fmt.Println(x, y, x+y)
}

// 关闭通道
v, ok := <-ch
如果通道接收不到数据后 ok 就为 false，这时通道就可以使用 close() 函数来关闭。
package main

import (
   "fmt"
)

func fibonacci(n int, c chan int) {
   x, y := 0, 1
   for i := 0; i < n; i++ {
      c <- x
      x, y = y, x+y
   }
   close(c)
}

func main() {
   c := make(chan int, 10)
   go fibonacci(cap(c), c)
   // range 函数遍历每个从通道接收到的数据，因为 c 在发送完 10 个
   // 数据之后就关闭了通道，所以这里我们 range 函数在接收到 10 个数据
   // 之后就结束了。如果上面的 c 通道不关闭，那么 range 函数就不
   // 会结束，从而在接收第 11 个数据的时候就阻塞了。
   for i := range c {  // 如果 c 不关闭, 这里会阻塞
      fmt.Println(i)
   }
}
// 输出结果
0 1 1 2 3 5 8 13 21 34
```



