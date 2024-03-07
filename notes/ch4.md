## 第 4 章--复合数据类型 容易混淆知识点
* 数组：(使用较少)
```Go
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
```Go
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
```Go
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
```Go
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
// ---------------- 匿名成员 --------------------------
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
```Go
// boolean     true
// number      -273.15
// string      "She said \"hello, 世界\""
// array       ["gold", "silver", "bronze"]
// object      {"year": 1990,
//              "event": "archery",
//              "medals": [["gold", "silver", "bronze"]]}
// (1) 把 Go 的数据结构转换为 JSON 称为 marshal, 通过 json.Marshal 实现。
type Movie struct {
  Title string
  Year  int   `json:"released"`         // 输出别名
  Color bool  `json:"color,omitempty"`  // omitempty 表示是零值或空值可以不解析
  Actors []string
}
// `json:"-"` 表示该字段不用于编码

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
// -------------------------------------------------------------------------------
// -------------------------------------------------------------------------------
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
