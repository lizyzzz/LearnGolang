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
