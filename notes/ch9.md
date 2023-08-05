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
