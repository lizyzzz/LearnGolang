## 第 2 章 容易混淆知识点
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
## 第 3 章容易混淆知识点
* 类型转换要显示进行，例如：int 和 int32 不是同一种类型，尽管它们可能都是32位来表示。使用时需要显式转换。
* 位运算中：& | ^ << >> ：其中 ^ 可以作为一元运算符表示 ~ (cpp 中 ~ 表示 按位非)，也可以作为二元运算符表示异或。
* 通常 Printf 的格式化字符串含有多个 % 谓词，这要求提供相同数量的操作数，而 % 后的副词 [1] 告知 Printf 使用第一个操作数。而 # 告知 Printf 输出相应的前缀。
```
o := 0666 // 八进制数
fmt.Printf("%d %[1]o %#[1]o\n", o) // 输出 438 666 0666
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x) // 输出 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

