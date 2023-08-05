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
