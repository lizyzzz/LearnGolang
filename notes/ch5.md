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
