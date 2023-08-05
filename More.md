## 第 2 章--程序结构 容易混淆知识点
* 实体第一个字母的大小写决定其可见性是否跨包。如果首字母大写，说明是对包外可见的，可以被包外的程序所引用，像fmt的Printf。
* new 函数：new(T) 返回一个 type 为 T 的指针，但是不能确定是在堆区还是栈区申请内存。
```
func f() *int {
  v := 1
  return &v
}

var p = f() // p 依然有效

// 这两个函数有着同样的行为
func newInt() *int {
  return new(int)
}
func newInt() *int {
  var dummy int
  return &dummy
}
```
* 对象的生命周期：包级别的变量的生命周期与整个程序的生命周期一样。局部变量的生命周期是动态的：即一直生存到它变得不可访问，这时占用的存储空间被回收。“不可访问”是指访问变量的路径（变量名，指针，引用等）不存在。在堆区或者栈区创建变量是由编译器决定，而不是由 var 或 new 决定。
```
// 这种情况称为 x 从 f 逃逸
var global *int
func f() {
  var x int
  x = 1
  global = &x // x 一定创建在堆区
}

func g() {
  y := new(int) // y 被创建在 栈区, 即使是 new 函数创建的
  *y = 1
}
```
* 垃圾回收机制对写出正确的代码很有帮助，但要注意变量的生命周期。
* 多重赋值问题：在实际更新变量前，右边所有的表达式被推演。
```
x, y = y, x // 交换 x y 的值
// 求最大公约数
func gcd(x, y int) int {
  for y != 0 {
    x, y = y, x % y
  }
}
```
* 短变量声明问题：短变量声明不需要声明所有在左边的变量。如果一些变量在同一个词法块中声明，那么对于这些变量，短声明等同于赋值。
```
in, err := os.Open(infile) // 声明 in err
// ...
out, err := os.Create(outfile) // 声明 out, 赋值 err
in, err := os.Create(outfile) // 错误, 没有新的变量
//---------------------------
特别注意: 只有在同一个词法块中已经存在变量的情况下, 短声明的行为才和赋值操作一样, 外层的声明将被忽略。
```
* 类型声明(别名)：如果名字是导出的(首字母大写)，其它包也可以访问.
```
// type name underlying-type
type Index int
type Loop int
// Index Loop 虽然底层都是int，但是是不同的类型
var i Index = 1
var l Loop = Loop(i) // 显式类型转换  
```
* 包的初始化
```
// (1) 包的初始化从初始化包级别的变量开始，这些变量按变量声明顺序初始化。
// (2) 如果有多个 go 文件，会按收到的文件的顺序进行：go工具会在调用编译器前将 .go 文件排序
// (3) 任何文件都可以包含任意数量 init 函数：
func init() {/* ... */}
// 这个 init 函数不能被调用和被引用，但它是个普通函数，在每一个文件里，当程序启动的时候，init 函数按照它们声明的顺序自动执行。
```
## 第 3 章--基本数据类型 容易混淆知识点
* 类型转换要显示进行，例如：int 和 int32 不是同一种类型，尽管它们可能都是32位来表示。使用时需要显式转换。
* 位运算中：& | ^ << >> ：其中 ^ 可以作为一元运算符表示 ~ (cpp 中 ~ 表示 按位非)，也可以作为二元运算符表示异或。
* 通常 Printf 的格式化字符串含有多个 % 谓词，这要求提供相同数量的操作数，而 % 后的副词 [1] 告知 Printf 使用第一个操作数。而 # 告知 Printf 输出相应的前缀。
```
o := 0666 // 八进制数
fmt.Printf("%d %[1]o %#[1]o\n", o) // 输出 438 666 0666
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x) // 输出 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```
* 字符串相关问题：GO语言的字符串用 UTF-8 编码。
```
// (1) 字符串的第 i 个字节不一定就是第 i 个字符，因为非 ASCII 字符的 UTF-8 码点需要两个字节来表示。
// (2) 字符串的单个字节指不能被修改，但可以整个字符串重新赋值。不可变意味着复制的字符串或子串共用一个内存。
s := "hello"
s[0] = 'L' // 编译错误
```
* 字符串字面量使用双引号 "" 或者反引号 ` 来创建。
```
双引号用来创建可解析的字符串，支持转义，但不能用来引用多行。-----双引号创建可解析的字符串应用最广泛
反引号用来创建原生的字符串字面量，可能由多行组成，但不支持转义，并且可以包含除了反引号外其他所有字符。-----反引号用来创建原生的字符串则多用于书写多行信息，HTML以及正则表达式。
```
* 变长UTF-8 编码
```
0~127： 0xxxxxxx 表示，首位是 0
128～2047：110xxxxx 10xxxxxx 表示，第一字节首位是 110，第二字节首位是 10
2048～65535：1110xxxx 10xxxxxx 10xxxxxx 表示，第一字节首位是 1110，第二字节首位是 10，第三字节首位是 10
有些不可以用键盘输入的字符，可以用转义序列表示
\uhhhh，其中 h 是16进制数，两个字节的转义
\xhh，表示的是一个字节转义
```
* 字符串与字节slice
```
// 4个标准包对字符串操作特别重要：bytes, strings, strconv, unicode
strings: 提供了函数，用于搜索，替换，比较，修整，切分与连接字符串
bytes: 也有类似函数，用于操作字节slice ([]byte 类型)
strconv: 提供的函数，用于转换布尔值，整数，浮点数为与之对应的字符串形式，或者把字符串转换为布尔值，整数，浮点数，或添加/去除引号
unicode: 具有判别文字符号值等特性的函数，如IsDigit、IsLetter、IsUpper等
// 字符串无法修改单个字节，但 字节slice 可以修改
s := "hello"
s[0] = 'L' // 编译错误
b := []byte(s) // 类型转换为 []byte, 并且会复制一份副本, 填入 s 含有的字节, 并生成一个 slice 引用指向整个数组
b[1] = 'a' // 正确
```
* bytes 包提供的 Buffer 类型可以高效处理 []byte
* 注意无类型常量的写法。
```
var f float64 = 212
fmt.Println((f - 32) * 5 / 9) // 100, (f - 32) * 5 的结果是 float64 型
fmt.Println(5 / 9 * (f - 32)) // 0, 5 / 9 的结果是 无类型整数 0
fmt.Println(5.0 / 9.0 * (f - 32)) // 0, 5.0 / 9.0 的结果是 无类型浮点数
```
## 第 4 章--复合数据类型 容易混淆知识点
* 数组：(使用较少)
```
// (1) 不同长度的数组是不同的类型。
// (2) 可以在初始化列表中使用下标的方式给指定下标的位置赋值
symbol := [...]int{99: -1} // symbol 的长度是 100, 第100位是 -1
// (3) 不确定长度的数组可以由列表初始化的个数决定。用 ... 表示
q := [...]int{3, 2, 1} // q 的长度是 3
// (4) 数组类型可比较的话, 数组也可比较
// (5) 数组作为函数参数是值传递 (与 cpp 不一样), 但也可以显式指定为指针传递。
func zero(val [32]byte) {
  // 值传递
}
func zero(ptr *[32]byte) {
  // 指针传递
}
```
* slice：(使用较多)
```
// (1) slice 有三个属性: 指针, 长度, 容量。指针指向底层数组的第一个可以从slice中访问的元素, 这个元素不一定是数组的第一个元素。
// (2) 一个底层数组可以对应多个 slice, 被不同的 slice 引用。
// (3) 当一个 slice 引用一个底层数组的子数组时, 引用不可以超过数组的边界, 但引用可以扩展 slice。
q1 := []string{1: "Jan", 12: "Dec"} // 长度为 13
summer := q1[6:9] // 长度为 3
fmt.Println(summer[:20]) // 宕机: 超出数组边界
endless := summer[:5]    // 在 slice 容量范围扩展了 slice
// (4) 注意 string 和 []byte 的区别，x[m:n]都返回原始字节的一个子序列，都是常量时间，引用方式也相同。区别在于 string 返回的是一个字符串, []byte返回的是一个字节slice
// (5) 因为 slice 包含了指向数组元素的指针, 所以将一个 slice 传递给函数时, 可以被修改。相当于创建了一个别名。
// (6) 不能对两个 slice 做比较, 标准库提供 bytes.Equal 来比较两个字节 slice，但其他类型的 slice 只能自己写函数来比较。
// (7) 值为 nil 的没有对应的底层数组，容量和长度都是 0, 但容量和长度都是 0 的 slice 不一定为 nil, 如 []int{}
// (8) make() 函数可以创建一个 slice
make([]T, len) // 引用整个数组, len == cap
make([]T, len, cap) // 引用 len 个, len <= cap
// (9) 使用 append 函数 将元素追加到 slice 后面。扩容策略与 cpp 的 vector 一样。
str := "hello"
s := make([]string, 0, 10)
s = append(s, str) // 这里的返回值底层数组可能发生变化，所以要接受返回值
s = append(s, str, str, str) // 这里的参数可以变化
// (10) slice 可以使用 reverse, rotate 等函数实现就地修改 slice 元素。
```
* map: 类型是 map[k]v， 也是引用类型
```
// (1) 键的类型必须可 == 比较。
ages := map[string]int {
  "alice": 31,
  "charlie": 34,
}
等价于
ages := make(map[string]int)
ages["alice"] = 31
ages["charlie"] = 34
ages := map[string]int{} // 新的空的map
// (2) 增改查删
ages["alice"] = 10 // 增改查
delete(ages, "alice") // 删
// (3) map 的顺序是不一定的
// (4) map 必须初始化才能使用
var ages map[string]int
ages["bob"] = 20 // 错误, 为 nil 值 map 赋值
// (5) map 不可比较
// (6) 当 key 不可比较时，可以自定义一个函数, 先把不可比较的 key 映射到唯一的可比较类型集合
```
* 结构体：
```
// (1) 结构体变量用 '.' 来访问属性，结构体指针也是用 '.' 来访问属性。
// (2) 成员变量的顺序影响结构体的同一性，顺序不一样结构体类型也不同。首字母大写的成员变量可导出(其他包可见)，首字母小写的不可导出。
// (3) 结构体初始化：1) 通过结构体字面量按顺序初始化；2) 通过变量名称和值来初始化。
/* 两种初始化方式不可混用, 也无法使用第一种初始化方式绕过不可导出的变量在其他包不可使用的原则 */
p := image.Point{1, 2}
p2 := image.Point{x: 1}
// (4) 结构体作函数参数时，通常用指针, 避免复制。
pp := &Point{1, 2} // 一种创建结构体指针的便捷方法
// (5) 如果结构体的成员可比较，那么结构体也可比较。
// (6) 结构体嵌套和匿名成员
type Point struct {
  X, Y int
}
type Circle struct {
  Center Point
  Raidus int
}
type Wheel struct {
  Circle Circle
  Spokes int
}

var w Wheel
w.Circle.Center.X  = 8 // 访问太长了
---------------- 匿名成员 --------------------------
type Circle struct {
  Point
  Radius int
}
type Wheel struct {
  Circle
  Spokes int
}
var w Wheel
w.X  = 8  // 等价于 w.Circle.Point.X
/* (即使 circle point 是不可导出的匿名成员，省略形式依然有效, 但注释中的写法无效) */
/* 不能通过结构体字面量的方式初始化 */
w = Wheel{8, 8, 5, 20}  // 错误
w = Wheel{X: 8, Y: 8, Radius: 5, Spokes: 20} // 错误
w = wheel{Circle{Point{8, 8}, 5}, 20} // 正确
w = wheel{
  Circle: Circle{
    Point: Point{8, 8},
    Radius: 5, // 逗号不能漏
  },
  Spokes: 20, // 逗号不能漏
} // 正确
```
* JSON (JavaScript对象表示法)
```
boolean     true
number      -273.15
string      "She said \"hello, 世界\""
array       ["gold", "silver", "bronze"]
object      {"year": 1990,
             "event": "archery",
             "medals": [["gold", "silver", "bronze"]]}
// (1) 把 Go 的数据结构转换为 JSON 称为 marshal, 通过 json.Marshal 实现。
type Movie struct {
  Title string
  Year  int   `json:"released"`         // 输出别名
  Color bool  `json:"color,omitempty"`  // omitempty 表示是零值或空值可以不解析
  Actors []string
}
var movies = []Movies {
  {Title: "Casablanca", Year: 1942, Color: false,
    Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
  {Title: "Cook Hand Luke", Year: 1967, Color: true,
    Actors: []string{"Paul Newman"}}
}

data, err := json.Marshal(movies)
if err != nil {
  Log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
/* 输出 不带任何空格*/
[{"Title":"Casablanca","released":1942,"Actors":["Humphrey Bogart",
"Ingrid Bergman"]},{"Title":"Cook Hand Luke","released":1967,"colo
r":true,"Actors":["Paul Newman"]}]
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
data, err := json.MarshalIndent(movies, "", "    ") // 每行输出的前缀, 缩进的字符串
if err != nil {
  Log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
/* 格式化输出 */
[
    {
        "Title": "Casablanca",
        "released": 1942,
        "Actors": [
            "Humphrey Bogart",
            "Ingrid Bergman"
        ]
    },
    {
        "Title": "Cook Hand Luke",
        "released": 1967,
        "color": true,
        "Actors": [
            "Paul Newman"
        ]
    }
]
// (2) 把 JSON 转换为 Go 的数据结构 称为 unmarshal, 通过 json.unMarshal 实现。(可以只解析对应字段)
var title []struct{ Title string }
if err := json.unMarshal(data, &titles); err != nil {
  Log.Fatalf("JSON unmarshaling failed: %s", err)
}
fmt.Println(titles) // [{Casablanca} {Cook Hand Luke}]
// (3) json.Encoder, json.Decoder 也可以实现对应的功能。
```
## 第 5 章--函数 容易混淆知识点
```
// (1) 函数参数没有默认参数的概念。
// (2) 函数返回值也可以先指定名字。(类似 matlab)
func sub(x, y int) (z int) {
  z = x - y
  return
}
// (3) 函数也可以没有函数体，说明该函数使用除了 GO 以外的语言实现。
// (4) Go 的函数调用栈可以随着使用而增长，可达到 1GB 左右的上限。
// (5) 错误处理策略:
    a. 把错误传递下去, 即直接返回
    b. 超出重试次数或时间再返回错误。
    c. 输出错误，优雅地停止程序。
    d. 利用日志记录错误，继续运行下去。
    e. 直接忽略整个错误日志。（很少用）
// (6) io.EOF 表示文件尾，这是一个错误。
// (7) 函数可以是变量类型 (类似 cpp 的函数指针), 且可以与 nil 比较
func square(n int) int {
  return n*n
}

f := square // f 的类型是 func(int) int
// (8) 匿名函数与函数闭包：注意捕获的变量。
func squares() func() int {
  var x int  // 逃逸 squares() 函数, 生命周期变长
  return func() int {
    x++
    return x * x
  }
}
func main {
  f := squares()
  fmt.Println(f()) // "1"
  fmt.Println(f()) // "4"
  fmt.Println(f()) // "9"
  fmt.Println(f()) // "16"
}
----------------------------------------
捕获迭代变量的坑
var rmdirs []func()
for _, d := range tempDirs() {
  dir := d  // 创建一个副本
  os.MkdirAll(dir, 0755)
  rmdirs = append(rmdirs, func() {
    os.RemoveAll(dir) // 捕获的是 dir 是内部变量
  })
}
/*  处理一些事情  */
for _, rmdir := range rmdirs {
  rmdir(); // 清理
}
// 如果换成下面的捕获错误
for _, dir := range tempDirs() {
  os.MkdirAll(dir, 0755)
  rmdirs = append(rmdirs, func() {
    os.RemoveAll(dir) // 捕获的是 dir 是共享变量--一个可访问的存储位置，而不是固定的值
  })
}
// (8) 变长函数 (即可变参数)
func sum(vals ...int) int {
  total := 0
  for _, val := range vals {
    total += val
  }
  return total
}

fmt.Println(sum()) // "0"
fmt.Println(sum(3)) // "3"
fmt.Println(sum(1, 2, 3, 4)) // "10"
//  slice 做可变参数
values := []int{1, 2, 3, 4}
sum(values...)
// (9) 延迟函数 defer
  a. 就是在一个普通的函数或方法调用，在调用之前加上关键字 defer。 : 实际的调用推迟到包含 defer 语句的函数结束后才执行
  b. 可以有多个 defer ，执行时以调用 defer 语句顺序的倒序执行。
var mu sync.Mutex
var m = make(map[string]int)
func lookup (key string) int {
  mu.Lock()
  defer mu.Unlock()
  return m[key]
}
// (10) 宕机与恢复
panic() 和 assert() 类似，但 panic() 可以接受非 bool 的参数
recover 函数： 只能在defer 函数里执行。终止当前的宕机状态并且返回宕机的值，函数不会从之前宕机的状态继续运行，而是正常返回。
func soleTitle(element int) (title string, err error) {
  type bailout struct{}
  defer func() {
    switch p := reover(); p {
      case nil:
        // 没有宕机
      case bailout{}:
        err = fmt.Errorf("预期的宕机")
      default:
        panic(p) // 未预期的宕机，继续宕机过程
    }
  }()

  /* do something with title */
  if title == "xxx" {
    panic(bailout{})
  }
  if title == "" {
    return "", fmt.Errorf("no title")
  }
  return title, nil
}
```
## 第 6 章--方法 容易混淆的知识点
* 方法声明
```
// (1) 方法名和成员变量名不能一样。
// (2) 与其他面向对象的语言不一样，Go 可以把方法绑定到任何类型上 (指针类型和接口类型除外)。
```
* 指针接受者的方法
```
// (1) 可以把接受者绑定到指针类型。
func (p *Point) ScaleBy(factor float64) {
  p.X *= factor
  p.Y *= factor
}
/* 调用方法 */
p := Point{1, 2}
pptr := &p
pptr.ScaleBy(2)
r := &Point{1, 2}
r.ScaleBy(2)
(&p).ScaleBy(2)
------------------
/* 但如果方法只要求一个 *Point 接受者 */
// 其中 p 是 Point 类型的 变量 。可以简写为
p.ScaleBy(3)
/* 编译器会对变量进行 &p 的隐式转换。只有变量才能这样做。
   不能对一个不能取地址的变量这样做。
   Point{1, 2}.ScaleBy(2) // 错误
*/
/* 同样方法只要求一个 Point 接受者  */
// 指针类型也可以隐式转换为 *pptr
pptr.Distance(q) // 对变量进行 *pptr 的隐式转换
// (2) 但本身就是指针类型的类型不能声明方法
type PtrInt *int
func (p PtrInt) f() { } // 编译错误
// (3) nil 是一个合法的接收值
```
* 结构体内嵌组成类型的方法
```
// (1) 内嵌了组成类型，组成类型的方法也可以调用。(并非继承，而是对象内嵌对象)
// (2) 匿名字段可以是个指向命名类型的指针。同样可以调用对应类型的方法。
// (3) 当有两个或以上内嵌字段，有同样的方法名时，编译器会报告错误。 
```
* 方法变量与方法表达式
```
// (1) 方法变量
p := Point{1, 2}
q := Point{4, 6}
distanceFromP := p.Distance  // 方法变量, 指定了接受者是 p
fmt.Println(distanceFromP(q)) // "5"
// (2) 方法表达式
distance = Point.Distance // 方法表达式, 类似 cpp bind() 函数绑定成员函数
fmt.Println(distance(p, q)) // 相当于调用 p.Distance(p), 第一个参数指定调用对象
```
* 封装
(1) Go 封装的级别是package, 而不是结构体，同一个 package 内, 结构体的成员都是可见的
(2) 结构体的成员首字母小写，则在另一个 package 不可见。
## 第 7 章--接口 容易混淆的知识点
一个接口类型定义了一套方法，如果一个具体类型要实现该接口，那么必须实现接口类型定义中的所有方法。  
使用时以接口类型做形参，具体类型做实参。注意：接口类型可以组合
* 实现接口
```
// (1) 类型 *T 实现了部分方法，但是类型 T 的变量可以直接调用 *T 的方法。这仅仅是个语法糖。
// 接口只能接受对应的 T 或 T* 类型。
type InSet struct {/*...*/}
func (* IntSet) String() string

var s IntSet
var _ = s.String() // 正确 (&s).String()
var _ fmt.Stringer = &s  // 正确
var _ fmt.Stringer = s   // 编译错误
// (2) 空接口类型可以接受任何值的赋值。但不能使用其中的值。需要一个方法还原其中的值。
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```
* 使用 flag.Value 接口来解释参数
```
package flag
/* 需要从命令行解析参数的类型实现该接口类型 */
type Value interface {
  String() string
  Set(string) err
}
fmt.CommandLine.Var(v Value, name string. usage string) // 以-name解析参数
// 使用方法例子 : ./bin -name val
```
* 接口值
一个接口类型的值（接口值）包括两个部分：一个具体类型和该具体类型的一个值。二者称为接口的动态类型和动态值。接口的零值就是动态类型和值都为 nil。
```
// (1) 一个接口值是否是 nil 取决于它的动态类型，可以用 == nil 或 != nil 来判断。调用一个 nil 接口的任何方法都会导致崩溃。
// (2) 接口值可以用 == 和 != 比较的，两接口值都为 nil 或 二者的动态类型完全一致且二者动态值相等。可以作为 map 的键值。
/* 值得注意的是 当动态类型一致时，但动态类型的值是不可比较的，那么这个比较方式会崩溃 */
// (3) 可以 %T 打印 接口的动态类型，利用反射（后续讲到）
// (4) 接口值为 nil 和 仅仅动态值为 nil 是不一样的。
```
* 类型断言
类型断言是一个作用在接口值上的操作，写出来类似于 x.(T), 其中 x 是一个接口类型的表达式，而 T 是一个类型(称为断言类型)。
```
// (1)如果断言类型 T 是一个具体类型，那么类型断言会检查 x 的动态类型是否 `就是` T 。
// 成功则返回 x 的动态值
var w io.Writer
w = os.Stdout
f := w.(*os.File) // 成功 f == os.Stdout(动态值)
c := w.(*bytes.Buffer) // 崩溃：接口持有的是 *os.File, 不是 *bytes.Buffer
// (2)如果断言类型 T 是一个接口类型，那么类型断言会检查 x 的动态类型是否 `满足` T 。
// 成功则返回一个接口值，接口值的类型和值部分没有变更。
var w io.Writer
w = os.Stdout
rw := w.(io.ReadWriter) // 成功, *os.File 有 Read 和 Write 方法, 也属于 io.ReadWriter 接口
w = new(ByteCounter)
rw = w.(io.ReadWriter) // 崩溃： *ByteCounter 没有 Read 办法
// (3) 如果操作数是一个空接口值，类型断言都失败。
// (4) 很少需要从一个接口类型向一个要求更宽松的类型做类型断言。
// (5) 使用两个返回值，不会直接崩溃。
f := w.(*os.File) // 成功 f == os.Stdout(动态值), ok == true
c := w.(*bytes.Buffer) // 失败 f == nil, ok == false
```
* 类型分支
```
/* 注意： 分支类型不允许使用 fallthrough */
switch x := x.(type) { // x 会被覆盖, 提取类型被下面复用
case nil: // ...
case int, uint: // ...
case bool:  // ...
case string: // ...
default:  // ...
}
```
## 第 8 章--goroutine和通道 容易混淆知识点
* 通道
通道是可以让一个 goroutine 发送特定值到另一个 goroutine 的通信机制。
```
// (1) 创建通道
ch := make(chan int)
ch := make(chan int, 3) // 带缓冲
ch <- x // 发送语句
x = <-ch // 接收语句并赋值，x 未被使用也是合法的。
// (2) 通道类型可以比较，引用同一份数据结构时 == ，否则 != , 也可以与 nil 比较。
// (3) 关闭后的发送操作将导致宕机，在已经关闭的通道上进行接收，会接收剩余的值，直到通道为空；再继续接收会收到通道元素类型的零值。
close(ch) // 关闭通道 
```
* 无缓冲通道
无缓冲通道上的发送操作将会阻塞，直到另一个 goroutine 在对应的通道上执行接收操作。
如果接收操作先执行，接收方 goroutine 将阻塞，直到另一个 goroutine 在同一个通道上发送值。
```
// (1) 使用无缓冲通道进行的通信导致发送和接收 goroutine 同步化。因此，无缓冲通道也称为同步通道。
func main {
  conn, err := net.Dial("tcp", "localhost:8000")
  if err != {
    log.Fatal(err)
  }
  done := make(chan struct{})
  go func() {
    io.Copy(os.Stdout, conn)
    log.Println("done")
    done <- struct{}{}
  }()
  mustCopy(conn, os.Stdin)
  conn.Close()
  <- done // 等待 go func() 结束
}
// (2) 通道也可以被垃圾回收机制回收。
```
* 管道
通道可以用来连接 goroutine，这样一个的输出是另一个的输入，就叫管道 (pipeline)。
```
// (1) 通知接收方通道是否关闭
go func() {
  for (
    x, ok := <- naturals
    if !ok {
      break // 通道关闭并读完
    }
    squares <- x * x
  )
  close(naturals)
}
/* 更普遍地，用 range */
go func() {
  for x := range naturals (
    squares <- x * x
  )
  close(naturals)
}
// (2) 试图关闭一个已经关闭的通道会到值宕机，就像关闭一个空通道一样。
```
* 单向通道类型
chan<- int 是一个只能发送 int 类型的通道。允许发送但不能接收。
<-chan int 是一个只能接收 int 类型的通道。允许接收但不能发送。
```
// 上述例子变化成
func counter(out chan<- int) {/* ... */}
func squarer(out chan<- int, in <-chan int) {/* ... */}
func printer(in <-chan int) {/* ... */}
func main() {
  naturals := make(chan int)
  squares := make(chan int)

  go counter(naturals)
  go squarer(squares, naturals)
  printer(squares)
}
/*
  注意： close 操作只能在发送方调用，在仅能接收的通道上调用，编译会报错
  在任何赋值操作中将双向通道转换为单向通道都是允许的，但反过来不行。
*/
```
* 缓冲通道
缓冲通道的大小由 make() 创建
```
// (1) 发送操作在队尾插入一个元素，接受操作从队头移除一个元素。
// 通道满时，发送操作会阻塞，通道为空时接收操作会阻塞。
// 可以用 cap(ch) 获得通道的容量, len(ch) 获得通道的元素个数但是意义不大，并发情况下很快失效。
// (2) goroutine 泄漏
/* 如果 resp 换成无缓冲通道，两个较慢的 goroutine 会被卡住，因为没有对应的 goroutine 接收，这时发生 goroutine 泄漏  */
func mirrorQuery() string {
  resp := make(chan string, 3)
  go func() { resp <- request("asia.gopl.io") }()
  go func() { resp <- request("europe.gopl.io") }()
  go func() { resp <- request("usa.gopl.io") }()
  return <-resp // 返回最早得到的镜像
}
// (3) 模拟令牌使用
var sema = make(chan struct{}, 10)
go func() {
  sema <- struct{} // 获取令牌
  /* do something... */
  <-sema  // 释放令牌
}
```
* 使用 select 多路复用
```
// (1) select 有一系列的情况和一个可选的默认分支。每一个情况指定一次 `通信`(在一些通道上进行发送和接收操作) 和关联的一段代码块。
// 当有多个通道准备好时，随机选择一个执行。
// 一个 select 只执行一次，select 可以嵌套在循环里实现循环监听。
select {
  case <-ch1:
    // ...
  case x := <-ch2:
    // ...
  case ch3 <- y: // 有 goroutine 接收时才会准备好
    // ...
  default:
    // ...
}
// (2) 当对一个缓冲区容量大于1时，在既有输入又有输出的情况不可确定。
for i := o; i < 10; i++ {
  select {
  case x := <-ch: 
    fmt.Println(x)
  case ch <- i:
  }
}
```
* 取消 goroutine 的广播机制
```
// (1) 当一个通道关闭且已读取完所有发送的值之后，接下来的接收操作立即返回，得到零值。
// 广播机制：不在通道上发送值，而是关闭它。
var done = make(chan struct{})
// 监测到有键盘输入时广播取消事件
go func() {
  os.Stdin.Read(make([]byte, 1))
  close(done)
}
for {
  select {
  case <-done:
    for range fileSizes {
      // 耗尽 fileSizes 通道中的元素
    }
    return
  case size, ok := <-fileSizes:
    // ...
  }
}
```
## 第 9 章--使用共享变量实现并发 容易混淆的知识点
* 锁
```
// (1) 类型 sync.Mutex
// 成员函数 Lock() 上锁, Unlock() 解锁.
// 通常使用 defer 搭配 Unlock() 使用.

// (2) 类型 sync.RWMutex (读写锁)
// 读锁: RLock() 上锁, RUnlock() 解锁
// 写锁: Lock() 上锁, Unlock() 解锁
```
* 内存宽松模型 (同 cpp)
* sync.Once (同 cpp 的 call_once() 函数 和 once_flag 变量)
sync.Once 包含一个 bool 变量 和 一个互斥量, 有唯一 Do() 方法
```
var loadIconsOnce sync.Once
var icons map[string]image.Image
func loadIcons() {/*...*/} // 初始化 icons 函数
// 并发安全的
func Icon(name string) image.Image {
  loadIconsOnce.Do(loadIcons) // 只调用一次
  return icons[name]
}
```
* goroutine 和 线程
```
// (1) 每个 os 线程有一个固定的大小的栈内存。一个 goroutine 在生命周期开始时只有一个很小的栈，典型情况为 2KB。
// goroutine 的栈不是固定大小的，它可以按需增大和缩小，大小限制可以达到 1GB。

// (2) os 线程由 os 内核调度，有上下文切换。
// goroutine 由 Go 运行时的一个调度器调度 (m:n, 调度m个goroutine到n个os线程)

// (3) Go 调度器使用 GOMAXPROCS 参数来确定需要使用多少个 os 线程。正在休眠或者正被通道通信阻塞的 goroutine 不需要占用线程
// 阻塞在 I/O 和其他系统调用中 或 调用非 Go 语言写的函数的 goroutine 需要一个独立的 os 线程，但这个线程不计算在 GOMAXPROCS 内。

// (4) goroutine 没有标识
```
## 第 10 章--包和 go 工具 容易混淆的知识点
![image-2](https://github.com/lizyzzz/LearnGolang/blob/main/images/2.png)
* 包
```
// (1) 导入声明可以给包重命名。防止同名，防止与变量同名
import (
  "crypto/rand"
  mrand "math/rand"
)
// (2) 空包导入(如果导入的包的名字没有在文件中使用，就会产生一个编译错误，但有时候我们仅仅是为了利用其副作用: 比如 init 函数)
import _ "image/png"
// (3) 包的导入路径对应存放目标文件的对应目录，也尽量对应存储源代码仓库的服务器的 URL。
// (4) 环境变量 GOPATH 是工作空间的根，可以修改环境变量的值指定工作空间。
// (5) 环境变量 GOROOT 指定 GO 发行版的根目录，其中提供所有标准库的包。用户无需设置 GOROOT。

// (6) 包的下载：包的导入路径不仅应指示本地工作空间中找到它的位置，还指明了通过互联网找到它的位置。
// go get 命令可以下载单一的包，也可以使用 ... 符号来下载子树或仓库。
go get gopl.io/...
// go get 创建的目录是远程仓库的真实客户端，即可以在对应路径下使用 git 命令。
// -u 参数会获取并更新本地的包到最新的版本。但对部署的项目很不友好，因为发布版本需要精准的版本控制。解决方案是家一层 vendor 目录，创建一份副本。

// (7) 包的构建：go build 命令编译每一个命令行参数中的包。如果包名是 main ，还会创建可执行文件。
// 可以通过全局路径或相对路径指定。
// 默认情况下，go build 命令构建所有需要的包以及它们所有的依赖，然后丢弃除了最终的可执行文件之外的所有编译后的代码
// go install 和 go build 非常相似，但 go install 会保存每一个包的编译代码和命令，而不会把它们丢弃。
// 编译后的包保存在 $GOPATH/pkg 目录中，对应于存放源文件的 src 目录, 可执行文件保存在 $GOPATH/bin 目录中。
// 后续再 go build 和 go install 对没有改变的包不再编译。
// go build -i 可以将包安装在独立于构建目标的地方。

// (8) 包的文档化: 可以用 go doc 查看包或包成员的文档。
go doc time

// (9) 内部包: 没有导出的标识符只能在同一个包内访问。
// go build 工具会特殊对待导入路径中包含路径片段 internal 的情况。这些包叫内部包。
// 内部包只能被另一个包导入，这个包位于以 internal 目录的父目录为根目录的树中。
net/http  // 可以导入 a
net/http/internal/chunked // a
net/http/httputil // 可以导入 a
net/url // 不可以导入 a

// (10) 包的查询: go list
// 判断一个包是否存在于工作空间中，如果存在输出它的导入路径
$ go list github.com/go-sql-driver/mysql
go list github.com/go-sql-driver/mysql
// go list ... 可以枚举工作空间所有包。
// go list gopl.io/ch3/... 枚举子树中所有的包
// go list ...xml... 枚举一个具体的主题
```
## 第 11 章--测试 容易混淆的知识点
* go test 测试工具
```
// (1) *_test.go 文件不是 go build 编译的目标, 是 go test 编译的目标。
// 在 *_test.go 文件中，三种函数需要特殊对待:
//  a. 功能测试函数：以 Test 前缀命名的函数，用来检测一些程序逻辑的正确性，go test 运行测试函数，并且报告结果是 PASS 还是 FAIL.
//  b. 基准测试函数：以 Benchmark 前缀命名的函数，用来测试某些操作的性能，go test 汇报操作的平均执行时间。
//  c. 示例函数：以 Example 前缀命名的函数，用来提供机器检查过的文档。
// (2) go test 工具扫描 *_test.go 文件 来寻找特殊函数，生成一个临时的 main 包来调用他们，然后编译和运行，并汇报结果，最后清空临时文件。
```
* 功能测试函数
```
// (1) 每一个测试文件必须导入 testing 包。这些函数的函数签名如下：
func Test'Name'(t *testint.T) { // 'Name' 是自己定义的名字，不用引号
  // ...
}
func TestSin(t *testint.T) {/*...*/}
func TestLog(t *testint.T) {/*...*/}
// (2) -v 参数可以输出每个测试用例的名称和运行时间。-run="grep"的参数是一个正则表达式，可以只运行与正则表达式匹配的函数。
// (3) 测试失败调用 t.Errorf 不会导致程序宕机，要是程序失败时结束，可以用t.Fatal。

// (4) 白盒测试: 测试者可以访问包的内部函数和数据结构，并且可以做一些常规用户无法做到的观察和改动。
// (5) 黑盒测试: 测试者对包的了解仅通过公开的 API 和文档，对包的内部逻辑时不透明的。
// (6) 外部测试：当需要循环引用测试时，定义一个同根目录下的外部测试包。
```
* 基准测试函数
相比 功能测试函数，增加了 b.N 用来指定检测操作的次数。
```
// (1) 每一个测试文件必须导入 testing 包。这些函数的函数签名如下：
func Benchmark'Name'(t *testint.B) { // 'Name' 是自己定义的名字，不用引号
  // ...
}
func BenchmarkSin(b *testint.B) {/*...*/}
// (2) 使用性能剖析
go test -cpuprofile=cpu.log
go test -blockprofile=block.log
go test -memprofile=mem.log
// (3) 使用 go tool pprof 来分析性能剖析结果。
go tool pprof -text -nodecount=10 ./http.test cpu.log 
```
* 实例函数
Example 函数既没有参数也没有结果，主要有三个目的：  
  a. 作为文档：以举例子的形式；
  b. 通过 go test 运行。
  c. 提供手动实验代码，(用处不大)。
## 第 12 章--反射 容易混淆的知识点
Go 语言提供了一种机制，在编译时不知道类型的情况下，可更新变量、在运行时查看值、调用方法以及直接对他们的布局进行操作，这种机制称为`反射`。  
反射功能由 reflect 包提供，定义了两个重要的类型： Type 和 Value.
```
// (1) reflect.Type : Type 表示 Go 语言的一个类型，它是一个有很多方法的接口，这些方法可以用来识别类型以及透视类型的组成部分。
// reflect.Type 接口只有一个实现，即类型描述符，接口值中的动态类型也是类型描述符。
// reflect.TypeOf 函数接受任何的 interface{} 参数，并且返回一个 reflect.Type--存储接口的动态类型。
t := reflect.TypeOf(3) // 一个 reflect.Type
fmt.Println(t.String()) // "int"
fmt.Println(t)          // "int"
var w io.Writer = os.Stdout
fmt.Println(reflect.TypeOf(w)) // "*os.File"
// reflect.Type 满足 Stringer

// (2) reflect.Value ： 可以包含任意类型的值。
// reflect,ValueOf 函数可以接受任何的 interface{} 参数，并将接口的动态值以 reflect.Value 的形式返回。
// 与 reflect.TypeOf 类似，但 reflect.ValueOf 返回的是具体值，不过 reflect.Value 也可以包含一个接口值。
v := reflect.ValueOf(3) // 一个 reflect.Value
fmt.Println(v) // "3"
fmt.Println("%v\n", v) // "3"
fmt.Println(v.String()) // 注意: "<int Value>" // 存储 int 类型的 Value 类型
// reflect.Value 也满足 fmt.Stringer.
// 调用 Value 的 Type 方法会把它的类型以 reflect.Type 返回.
t := v.Type() // 一个 reflect.Type
fmt.Println(t.String()) // "int"
// reflect.ValueOf 的逆操作是 reflect.Value.Interface 方法, 它返回一个 interface{} 接口值, 与 reflect.Value 包含同一个具体值。
v := reflect.ValueOf(3) // 一个 reflect.Value
x := v.Interface() // 一个 interface{}
i := x.(int)       // 一个 int
fmt.Println("%d\n", i) // "3"
// Value 和 interface{} 的区别:
//  a. 空接口隐藏了值的布局信息、内置操作和相关方法, 只提供了一些公共的方法
//  b. Value 有很多方法可以用来分析所包含的值，而不用知道它的类型。
// Value 的 Kind 方法 可以区分不同的类型
v.Kind() // 有以下类型
reflect.Int reflect.Int8 ... reflect.Int64
reflect.Uint reflect.Uint8 ... reflect.Uint64 reflect.Uintptr
reflect.Bool
reflect.String
reflect.chan reflect.Func reflect.Ptr reflect.Slice reflect.Map
// 省略了浮点数和复数类型没写出来。

// (3) 使用 reflect.Value 来设置值
// 一个变量是一个可寻址的存储区域，其中包含了一个值，这里的可寻址是指`它的值可以通过这个地址来更新`。
// reflect.Value 类型某些是可寻址的，某些是不可寻址的。
x := 2                    // 值    类型   可寻址？
a := reflect.ValueOf(2)   // 2     int   no
b := reflect.ValueOf(x)   // 2     int   no
c := reflect.ValueOf(&x)  // &x    *int  no
d := c.Elem()             // 2     int   yes(x)
// a 是不可寻址的原因是，它包含的仅仅是整数 2 的一个副本。 b c 也是
// 通过 reflect.ValueOf(x) 返回的 reflect.Value 都是不可寻址的
// 但 d 是通过对 c 中的指针解引用得来的，所以它是可寻址的。(Elem 函数返回指针指向的变量，但是是以 reflect.Value 返回)
// 可以通过调用 reflect.ValueOf(&x).Elem() 来获得任意变量 x 可寻址的 Value 值。
// 返回的是指针的解引用的 Value 都是可寻址的，比如 reflect.ValueOf(e).Index(i) /* 其中 e 是一个 slice*/
x := 2
d := reflect.ValueOf(&x).Elem()   // d 代表变量 x
px := d.Addr().Interface().(*int) // px := &x, 前提是指导 d 是 int
*px = 3                           // x = 3
fmt.Println(x)                    // 3
d.Set(reflect.ValueOf(4))         // 可直接调用 Set, (不可寻址的变量使用 Set 也会崩溃)
d.Set(reflect.ValueOf(int64(4)))  // 崩溃, 必须注意原来的类型
fmt.Println(x)                    // 4
// 另外，利用反射可以读取到未导出结构字段的值，常规方法是不可以的，但不能用反射更新这些值。
// Value.CanAddr() 可以返回是否可寻址, Value.CanSet() 返回是否可修改。

// (4) 注意事项：谨慎使用
// a. 基于反射的代码是很脆弱的，稍有不慎很容易导致崩溃。
// b. 反射的相关操作无法做静态类型检查，大量使用反射难以理解。
// c. 基于反射的函数会比为特定类型优化的函数慢一两个数量级。
```
## 第 13 章--低级编程 容易混淆的知识点
unsafe 包广泛使用在和操作系统交互的低级包(比如 runtime, os, syscall 和 net) 中。
```
// (1) unsafe.Sizeof 返回指定参数在内存中占用的字节长度。
// 在 32位系统上，1 字 == 4 字节；在 64位操作系统上，1 字 == 8 字节
// 结构体也会存在内存对齐问题。
/*   --------- 类型的字节长度 ----------------    */
bool                             1 字节
intN, uintN, floatN, complexN    N/8 个字节
int, uint, uintptr               1 字
*T                               1 字
string                           2 字(数组指针, 数组长度)
[]T                              3 字(数组指针, 长度, 容量)
map                              1 字
func                             1 字
chan                             1 字
interface                        2 字(动态类型, 动态值)

// (2) unsafe.Alignof 报告参数类型所要求的对齐方式
// (3) unsafe.Offsetof 报告成员 f 相对于结构体 x 起始地址偏移值，如果有内存空位，也计算在内。

// (4) unsafe.Pointer 相当于 cpp 的 void* 可以对人以指针类型转化，但转换类型可能会变化。

// (5) Go 也可以调用 C/C++ 代码(用 cgo) (不常用的功能)
import "C" // 虽然没有包的名字为 C
```

















  


