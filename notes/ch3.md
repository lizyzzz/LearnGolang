## 第 3 章--基本数据类型 容易混淆知识点
* 类型转换要显示进行，例如：int 和 int32 不是同一种类型，尽管它们可能都是32位来表示。使用时需要显式转换。
* 位运算中：& | ^ << >> ：其中 ^ 可以作为一元运算符表示 ~ (cpp 中 ~ 表示 按位非)，也可以作为二元运算符表示异或。
* 通常 Printf 的格式化字符串含有多个 % 谓词，这要求提供相同数量的操作数，而 % 后的副词 [1] 告知 Printf 使用第一个操作数。而 # 告知 Printf 输出相应的前缀。
```Go
o := 0666 // 八进制数
fmt.Printf("%d %[1]o %#[1]o\n", o) // 输出 438 666 0666
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x) // 输出 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```
* 字符串相关问题：GO语言的字符串用 UTF-8 编码。
```Go
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
2048～65535：11110xxx 10xxxxxx 10xxxxxx 10xxxxxx 表示，第一字节首位是 11110，第二字节首位是 10，第三字节首位是 10, 第四字节首位是 10
有些不可以用键盘输入的字符，可以用转义序列表示
\uhhhh，其中 h 是16进制数，两个字节的转义
\xhh，表示的是一个字节转义
```
* 字符串与字节slice
```Go
// 4个标准包对字符串操作特别重要：bytes, strings, strconv, unicode
// strings: 提供了函数，用于搜索，替换，比较，修整，切分与连接字符串
// bytes: 也有类似函数，用于操作字节slice ([]byte 类型)
// strconv: 提供的函数，用于转换布尔值，整数，浮点数为与之对应的字符串形式，或者把字符串转换为布尔值，整数，浮点数，或添加/去除引号
// unicode: 具有判别文字符号值等特性的函数，如IsDigit、IsLetter、IsUpper等
// 字符串无法修改单个字节，但 字节slice 可以修改
s := "hello"
s[0] = 'L' // 编译错误
b := []byte(s) // 类型转换为 []byte, 并且会复制一份副本, 填入 s 含有的字节, 并生成一个 slice 引用指向整个数组
b[1] = 'a' // 正确
// for range string 的用法
str2 := "Chinese: 中国话"
for index, r := range str2 {
  fmt.Printf("character %c starts at byte position %d\n", r, index)
}
for index, r := range str2 {
  fmt.Printf("%-2d    %d    %U    '%c'    % X \n", index, r, r, r, []byte(string(rune)))
}
/* 输出
character C starts at byte position 0
character h starts at byte position 1
character i starts at byte position 2
character n starts at byte position 3
character e starts at byte position 4
character s starts at byte position 5
character e starts at byte position 6
character : starts at byte position 7
character   starts at byte position 8
character 中 starts at byte position 9
character 国 starts at byte position 12
character 话 starts at byte position 15
0     67    U+0043    'C'    43 
1     104    U+0068    'h'    68 
2     105    U+0069    'i'    69 
3     110    U+006E    'n'    6E 
4     101    U+0065    'e'    65 
5     115    U+0073    's'    73 
6     101    U+0065    'e'    65 
7     58    U+003A    ':'    3A 
8     32    U+0020    ' '    20 
9     20013    U+4E2D    '中'    E4 B8 AD 
12    22269    U+56FD    '国'    E5 9B BD 
15    35805    U+8BDD    '话'    E8 AF 9D
*/
```
* bytes 包提供的 Buffer 类型可以高效处理 []byte
* 注意无类型常量的写法。
```Go
var f float64 = 212
fmt.Println((f - 32) * 5 / 9) // 100, (f - 32) * 5 的结果是 float64 型
fmt.Println(5 / 9 * (f - 32)) // 0, 5 / 9 的结果是 无类型整数 0
fmt.Println(5.0 / 9.0 * (f - 32)) // 100, 5.0 / 9.0 的结果是 无类型浮点数
```
