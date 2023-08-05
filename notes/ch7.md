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
