# Go 小抄

后面带 * 的都需要引入对应的包。

# 索引
1. [基础语法](#基础语法)
2. [操作符](#操作符)
    * [运算符](#arithmetic)
    * [Comparison](#comparison)
    * [Logical](#logical)
    * [Other](#other)
3. [声明](#声明)
    * [枚举](#枚举)
4. [函数](#函数)
    * [函数值](#函数值)
    * [匿名函数](#匿名函数)
    * [闭包](#闭包)
    * [可变参数](#可变参数)
    * [defer](#defer)
5. [数据类型](#数据类型)
    * [基础数据类型](#基础数据类型)
    * [自定义类型](#自定义类型)
    * [引用类型](#引用类型)
6. [类型转换](#类型转换)
7. [Packages](#packages)
8. [流程控制](#流程控制)
    * [If](#if)
    * [Loops](#loops)
        * [键值循环](#键值循环)
        * ~~fallthrough~~
        * [goto](#goto)
        * [break](#break)
        * [continue](#continue)
    * [Switch](#switch)
9. [数组，切片，范围](#数组-切片-范围)
    * [数组](#数组)
    * [切片](#切片)
    * [数组和切片的操作](#数组和切片的操作)
10. [映射，列表](#映射-列表)
    * [映射](#映射)
    * [sync.Map*](#sync.Map)
    * [列表*](#列表)
11. [结构体](#结构体)
    * [匿名结构体](#匿名结构体)
    * [类型内嵌和结构体内嵌](#类型内嵌和结构体内嵌)
12. [指针](#指针)
13. [接口](#接口)
14. [嵌入](#嵌入)
15. [Errors](#errors)
16. [Concurrency](#concurrency)
    * [Goroutines](#goroutines)
    * [Channels](#channels)
    * [Channel Axioms](#channel-axioms)
17. [Printing](#printing)
18. [Reflection](#reflection)
    * [Type Switch](#type-switch)
    * [Examples](https://github.com/a8m/reflect-examples)
19. [Snippets](#snippets)
    * [Http-Server](#http-server)
    * [函数封装结构体的初始化过](#函数封装结构体的初始化过程)
    * [使用匿名结构体](#使用匿名结构体)
    * [闭包实现生成器](#playergen)

## 感谢

多数代码摘自 [A Tour of Go](http://tour.golang.org/)，这是一个很好的 Go 入门站点。如果你是第一次接触 Go，建议你可以先去试验下。

我在保留英文版内容的基础上，补充了《Go 语言从入门到进阶实战》中的一些代码。

## Go 特性概览

* 命令式编程
* 静态类型
* 类似 C 的语法结构（但没有括号和分号）和 Oberon-2 的结构
* 编译成机器码（没有虚拟机）
* 没有类，但是有带方法的结构体
* 接口
* 没有继承。有 [类型嵌入](http://golang.org/doc/effective%5Fgo.html#embedding)
* 函数是一等公民
* 函数可以有多个返回值
* 有闭包
* 有指针，但不支持指针运算
* 原生支持并发：Goroutines 和 Channels

# 基础语法

## Hello World
File `hello.go`:
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello Go")
}
```
`$ go run hello.go`

## 操作符
### 运算符
|Operator|Description|
|--------|-----------|
|`+`|addition|
|`-`|subtraction|
|`*`|multiplication|
|`/`|quotient|
|`%`|remainder|
|`&`|bitwise and|
|`\|`|bitwise or|
|`^`|bitwise xor|
|`&^`|bit clear (and not)|
|`<<`|left shift|
|`>>`|right shift|

### Comparison
|Operator|Description|
|--------|-----------|
|`==`|equal|
|`!=`|not equal|
|`<`|less than|
|`<=`|less than or equal|
|`>`|greater than|
|`>=`|greater than or equal|

### Logical
|Operator|Description|
|--------|-----------|
|`&&`|logical and|
|`\|\|`|logical or|
|`!`|logical not|

### Other
|Operator|Description|
|--------|-----------|
|`&`|address of / create pointer|
|`*`|dereference pointer|
|`<-`|send / receive operator (see 'Channels' below)|

## 声明
类型在标识符后面！
```go
var foo int // declaration without initialization
var foo int = 42 // declaration with initialization
var foo, bar int = 42, 1302 // declare and init multiple vars at once
var foo = 42 // type omitted, will be inferred
foo := 42 // shorthand, only in func bodies, omit var keyword, type is always implicit
const constant = "This is a constant"
a, _ = GetData() // _ is anonymous variable, 不占用命名空间也不分配内存
```

### 枚举

```go
// iota can be used for incrementing numbers, starting from 0
const (
    _ = iota
    a
    b
    c = 1 << iota // 左移一位
    d
)
fmt.Println(a, b) // 1 2 (0 is skipped)
fmt.Println(c, d) // 8 16 (2^3, 2^4)
```

```go
type Weapon int

const (
	Arrow Weapon = iota
	Shuriken
	SniperRifle
	Rifle
	Blower
)

func main() {
	fmt.Println(Arrow, Blower) // 0 4

	var weapon Weapon = Blower
	fmt.Println(weapon) // 1
}
```

## 函数
```go
// a simple function
func functionName() {}

// function with parameters (again, types go after identifiers)
func functionName(param1 string, param2 int) {}

// multiple parameters of the same type
func functionName(param1, param2 int) {}

// return type declaration
func functionName() int {
    return 42
}

// Can return multiple values at once
func returnMulti() (int, string) {
    return 42, "foobar"
}
var x, str = returnMulti()

// Return multiple named results simply by return
func returnMulti2() (n int, s string) {
    n = 42
    s = "foobar"
    // n and s will be returned
    return
}
var x, str = returnMulti2()

```

### 函数值
函数像其他值一样，拥有类型，可以被赋值给其他变量，传递给函数，从函数返回。

```go
func fire() bool {
}
// 注意声明一个函数变量的时候，变量的类型签名和函数签名要一致，也就是说返回值也要属于类型的一部分。
var f func() bool
f = fire
f()
```

### 匿名函数
变量的生命周期不由它的作用域决定。

```go
func main() {
    // assign a function to a name
    add := func(a, b int) int {
        return a + b
    }
    // use the name to call the function
    fmt.Println(add(3, 4))
}}

// 匿名函数用作回调函数
func visit(list []int, f func(int)) {
    for _, v := range list {
        f(v)
    }
}
func main() {
    visit([]{1, 2, 3, 4}, func(v int) {
        fmt.Println(v)
    })
}
```

### 闭包
引用了外部变量的匿名函数就是闭包。函数类型就像结构体一样，可以被实例化。函数是编译器静态的概念，闭包是运行期动态的概念。
```go
// Closures, lexically scoped: Functions can access values that were
// in scope when defining the function
func scope() func() int{
    outer_var := 2
    foo := func() int { return outer_var}
    return foo
}

func another_scope() func() int{
    // won't compile because outer_var and foo not defined in this scope
    outer_var = 444
    return foo
}


// Closures
func outer() (func() int, int) {
    outer_var := 2
    inner := func() int {
        outer_var += 99 // outer_var from outer scope is mutated.
        return outer_var
    }
    inner()
    return inner, outer_var // return inner func and mutated outer_var 101
}
```

### 可变参数
```go
// func 函数名(固定参数列表, v ... T) (返回参数列表) - v 为可变参数类型为 []T, T 为 interface{} 时传入的可以是任意类型
func main() {
	fmt.Println(adder(1, 2, 3)) 	// 6
	fmt.Println(adder(9, 9))	// 18

	nums := []int{10, 20, 30}
	fmt.Println(adder(nums...))	// 60
}

// By using ... before the type name of the last parameter you can indicate that it takes zero or more of those parameters.
// The function is invoked like any other function except we can pass as many arguments as we want.
func adder(args ...int) int {
	total := 0
	for _, v := range args { // Iterates over the arguments whatever the number.
		total += v
	}
	return total
}

// 在多个可变参数函数中传递参数，在传递时在可变参数变量后添加 ... 即可
func rawPrint(rawList ...interface{}) {
	for _, a := range rawList {
		fmt.Println(a)
	}
}
func print(slist ...interface{}) {
	rawPrint(slist...)
}
func main() {
	print(1, 2, 3)
}
```

### defer
用来延迟执行语句在函数退出时释放资源，比如打开和关闭文件、接收和回复请求、加锁和解锁。

## 数据类型
### 基础数据类型
```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8 的别名，代表了 ASCII 码的一个字符串

rune // int32 的别名 ~= 代表了一个 UTF-8 字符 (Unicode code point)

float32 float64 // IEEE 754

complex64 complex128
```

### 自定义类型
type 可以将各种基本类型定义为自定义类型。
```go
type MyInt int
var m MyInt = 1
```
### 引用类型
只有三种引用类型：slice(切片)、map(字典)、channel(管道)。


## 类型转换
```go
// T(表达式) - T 代表要转换的类型，表达式包括变量、复杂算子和函数返回值等
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)

// alternative syntax
i := 42
f := float64(i)
u := uint(f)
```

## Packages
* Package declaration at top of every source file
* Executables are in package `main`
* Convention: package name == last name of import path (import path `math/rand` => package `rand`)
* Upper case identifier: exported (visible from other packages)
* Lower case identifier: private (not visible from other packages)

## 流程控制

### If
```go
func main() {
	// Basic one
	if x > 10 {
		return x
	} else if x == 10 {
		return 10
	} else {
		return -x
	}

	// You can put one statement before the condition
	if a := b + c; a < 42 {
		return a
	} else {
		return a - 42
	}

	// Type assertion inside if
	var val interface{}
	val = "foo"
	if str, ok := val.(string); ok {
		fmt.Println(str)
    }

    // example 2
    if err := Connect(); err != nil {
        fmt.Println(err)
        return
    }
}
```

### Loops
```go
    // There's only `for`, no `while`, no `until`
    for i := 1; i < 10; i++ {
    }
    for ; i < 10;  { // while - loop
    }
    for i < 10  { // you can omit semicolons if there is only a condition
    }
    for { // you can omit the condition ~ while (true)
    }

    // use break/continue on current loop
    // use break/continue with label on outer loop
here:
    for i := 0; i < 2; i++ {
        for j := i + 1; j < 3; j++ {
            if i == 0 {
                continue here
            }
            fmt.Println(j)
            if j == 2 {
                break
            }
        }
    }

there:
    for i := 0; i < 2; i++ {
        for j := i + 1; j < 3; j++ {
            if j == 1 {
                continue
            }
            fmt.Println(j)
            if j == 2 {
                break there
            }
        }
    }
```

#### 键值循环
for range 遍历数组、切片、字符串、map 以及通道 channel。

```go
for key, value := range []int{1, 2, 3, 4} {
}

for key, value := range "hello" {
}

for key, value := range map[string]int{ "hello": 100, "world": 200 } {
}

c := make(chan int)
go func() {
    c <- 1
    c <- 2
    c <- 3
    close(c)
}()
for v := range c {
    fmt.Println(v)
}
```

#### goto
快速跳出循环，避免重复。

```go
	err := firstCheckError()
	if err != nil {
		goto onExit
	}
	err = secondCheckError()
	if err != nil {
		goto onExit
	}
	fmt.Println("Done")
	return
onExit:
	fmt.Println(err)
	exitProcess()
```

#### break
可以结束 for、swith、select 的代码块。break 后面添加标签，表示退出某个标签对应的代码块。

```go
OuterLoop:
	for i := 0; i < 2; i++ {
		for j := 0; j < 5; j++ {
			switch j {
			case 2:
				fmt.Println(i, j)
				break OuterLoop // 跳出指定循环
			case 3:
				fmt.Println(i, j)
				break OuterLoop
			}
		}
	}
```

#### continue
结束当前循环，开始下一次循环迭代过程，仅限在 for 循环内使用。若 continue 后添加标签，表示开始标签对应的循环。

```go
OuterLoop:
	for i := 0; i < 2; i++ {
		for j := 0; j < 5; j++ {
			switch j {
			case 2:
				fmt.Println(i, j)
				continue OuterLoop // 开始下一次外层循环，i 变为 1
			}
		}
	}
```

### Switch
```go
// switch statement
switch operatingSystem {
case "darwin":
    fmt.Println("Mac OS Hipster")
    // cases break automatically, no fallthrough by default
case "linux":
    fmt.Println("Linux Geek")
default:
    // Windows, BSD, ...
    fmt.Println("Other")
}

// as with for and if, you can have an assignment statement before the switch value
switch os := runtime.GOOS; os {
case "darwin": ...
}

// you can also make comparisons in switch cases
number := 42
switch {
    case number < 42:
        fmt.Println("Smaller")
    case number == 42:
        fmt.Println("Equal")
    case number > 42:
        fmt.Println("Greater")
}

// cases can be presented in comma-separated lists
var char byte = '?'
switch char {
    case ' ', '?', '&', '=', '#', '+', '%':
        fmt.Println("Should escape")
}
```

## 数组-切片-范围

### 数组
数组是一段固定长度的连续内存区域。可以修改数组成员，数组大小不可变化。
```go
// var 数组变量名 [元素数量]T - T 可以是任意基本类型，也可以是数组以实现多维数组
var a [10]int // declare an int array with length 10. Array length is part of the type!
a[3] = 42     // set elements
i := a[3]     // read elements

// declare and initialize
var a = [2]int{1, 2}
a := [2]int{1, 2} // shorthand
a := [...]int{1, 2} // `...` 表示让编译器确定数组大小
```

### 切片
切片是一个拥有相同类型元素的可变长度的序列。
```go
// var name []T - T 代表切片元素类型，可以是整型、浮点型、布尔型、切片、map、函数等
var a []int                              // declare a slice - similar to an array, but length is unspecified，默认值是 nil
var a = []int {1, 2, 3, 4}               // declare and initialize a slice (backed by the array given implicitly)
a := []int{1, 2, 3, 4}                   // shorthand
chars := []string{0:"a", 2:"c", 1: "b"}  // ["a", "b", "c"]

// slice[开始位置:结束位置] - 取出元素不包含结束位置
var b = a[lo:hi]	// creates a slice (view of the array) from index lo to hi-1
var b = a[1:4]		// slice from index 1 to 3
var b = a[:3]		// missing low index implies 0
var b = a[3:]		// missing high index implies len(a)
a = append(a, 17, 3)	// append items to slice a，容量扩容规律按容量的 2 倍数扩充，1、2、4、8、16
c := append(a, b...)	// concatenate slices a and b

// make([]T, size, cap) - T 切片元素的类型，size 是为这个类型分配多少个元素，cap 预分配的元素数量只是提前分配空间优化性能。
a = make([]byte, 5, 5) // first arg length, second capacity
a = make([]byte, 5) // capacity is optional

// create a slice from an array
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // a slice referencing the storage of x
```

### 数组和切片的操作
`len(a)` gives you the length of an array/a slice. It's a built-in function, not a attribute/method on the array.

```go
// loop over an array/a slice
for i, e := range a {
    // i is the index, e the element
}

// if you only need e:
for _, e := range a {
    // e is the element
}

// ...and if you only need the index
for i := range a {
}

// In Go pre-1.4, you'll get a compiler error if you're not using i and e.
// Go 1.4 introduced a variable-free form, so that you can do this
for range time.Tick(time.Second) {
    // do it once a sec
}
```

`copy()` 复制切片元素到另一个切片

```go
// copy(destSlice, srcSlice []T) int - srcSlice 为数据来源切片，srcSlice 为复制目标
srcData := []int{1, 2, 3, 4, 5, 6}
copyData := make([]int, 7)
copy(copyData, srcData)
fmt.Println(copyData) // [1 2 3 4 5 6 0]
srcData[0] = 10
copy(copyData, srcData[0:2]) // 复制是从 destSlice 的起始位置开始替换
fmt.Println(copyData)  // [10 2 3 4 5 6 0]
```

## 映射-列表

### 映射

```go
// map[KeyType]ValueType - KeyType 是键的类型 ValueType 是键对应值的类型
m = make(map[string]int)
m["key"] = 42
fmt.Println(m["key"])
fmt.Println(m["none"]) // 0，查找一个不存在的键，返回的是 ValueType 的默认值

delete(m, "key")

elem, ok := m["key"] // test if key "key" is present and retrieve it, if so

// map literal，声明时填充内容
var m = map[string]Vertex{
    "Bell Labs": {40.68433, -74.39967},
    "Google":    {37.42202, -122.08408},
}

// iterate over map content
for key, value := range m {
}
```

### sync.Map
map 在并发情况下，只读是线程安全，同时读写线程不安全。故需要使用 sync.Map

```go
var scene sync.Map
scene.Store("greece", 97) // 不能使用 map 的方式取值和设置，使用 sync.Map 的方式
scene.Store("london", 100) // Store 表示存储
fmt.Println(scene.Load("london")) // Load 表示获取
scene.Delete("london") // Delete 表示删除

// 遍历需要提供一个匿名函数
scene.Range(func(k, v interface{}) bool {
  fmt.Println("iterate:", k, v)
  return
})
```

### 列表
列表是一种非连续存储的容器，由多个节点组成，节点通过一些变量记录彼此间的关系。Go 使用双链表来实现列表以高效地进行任意位置的元素插入和删除操作。

列表没有具体元素类型的限制。给一个列表放入了非期望类型的值，取出值后，将 `interfact{}` 转换为期望类型会发生宕机。
```go
import "container/list"
// 变量名 := list.New() 或 var 变量名 list.List
l := list.New()
element := l.PushBack("first") // 尾部添加元素后返回 *list.Element 结构
l.PushFront(67) // 头部添加元素后返回 *list.Element 结构
l.InsertAfter("high", element) // 在 element 前添加 high
l.InsertBefore("noon", element) // 在 element 后添加 noon
l.Remove(element)

for i := l.Front(); i != nil; i = i.Next() {
    fmt.Println(i.Value)
}
```

## 结构体
Go 没有类，也不支持基于类的继承等面向对象的概念。只有结构体，结构体可以拥有自己的方法。
```go
// A struct is a type. It's also a collection of fields

// Declaration
type Vertex struct {
    X, Y int
}

// Creating
var v Vertex // var ins T， ins 的类型是 T，属于值类型
v := new(Vertex) // ins := new(T)，ins 的类型是 *T，属于指针
v := &Vertex{} // ins := &T{}，ins 的类型是 *T，属于指针，这是最常用的方法

// 初始化结构体的成员变量
var v = Vertex{1, 2}
var v = Vertex{X: 1, Y: 2} // Creates a struct by defining values with keys
var v = []Vertex{{1,2},{5,2},{5,5}} // Initialize a slice of structs

// Accessing members
v.X = 4

// func(接收器变量 接收器类型) 方法名(参数列表) (返回参数) {} - 接收器类型可以是指针类型和值类型
func (v *Vertex) setX(x int) {
    v.x = x
}
func (v *Vertex) getX() int {
    return v.x
}
func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// Call method
v.Abs()

// For mutating methods, you need to use a pointer (see below) to the Struct
// as the type. With this, the struct value is not copied for the method call.
func (v *Vertex) add(n float64) {
    v.X += n
    v.Y += n
}
```
### 匿名结构体
Cheaper and safer than using `map[string]interface{}`.
```go
point := struct {
	X, Y int
}{1, 2} // 可以不初始化成员
```
### 类型内嵌和结构体内嵌
```go
// 类型内嵌
type Data struct {
    int
    float32
    bool
}
ins := &Data{
    int: 10,
    float32: 3.14,
    bool: true,
}

// 结构体内嵌
type BasicColor struct {
    R, G, B float32
}
type Color struct {
    BasicColor
    Alpha float32
}
```

## 指针
分为两个核心概念：
1. 类型指针，允许对这个指针类型的数据进行修改。传递数据使用指针，而无须拷贝数据。类型指针不能进行偏移和运算。
2. 切片，由指向起始元素的原始指针、元素数量和容量组成。

使用 `new(类型)` 和 `&` 创建创建出来的对象均为指针。
```go
p := Vertex{1, 2}  // p is a Vertex
q := &p            // q is a pointer to a Vertex，`&` 操作符进行取地址操作
r := &Vertex{1, 2} // r is also a pointer to a Vertex，r 的类型是 *Vertex
value := *r        // value is `Vertex{1, 2}`, `*` 是从指针取值操作
*r = Vertex{4, 6}  // * 操作符的根本意义就是操作指针指向的变量，在右侧是取变量的值，在左侧就是将值设置给指向的变量

// Vertex 的指针类型是 *Vertex
var s *Vertex = new(Vertex) // new(类型) 是创建指针的另外一种方法，指针指向的值为默认值
```

## 接口
```go
// interface declaration
type Awesomizer interface {
    Awesomize() string
}

// types do *not* declare to implement interfaces
type Foo struct {}

// instead, types implicitly satisfy an interface if they implement all required methods
// Way 1:
// func (foo Foo) Awesomize() string {
//     return "Awesome!"
// }
// Way 2:
func (foo *Foo) Awesomize() string {
    return "Awesome!"
}
```

1. 函数的声明不能直接实现接口，需要将函数定义为类型后，使用类型实现结构体。当类型方法被调用时，还需要调用函数本身。
2. 函数无需被实例化，只需将函数转换为对应函数类型即可。
3. 函数来源可以是命名函数、匿名函数或闭包。
4. **函数无法实现不带参数的接口方法。**
5. **函数无法实现有返回值的接口方法。**

```go
type Awesomizer interface {
	Awesomize(s string)
}

type FuncCaller func(s string)

func (f FuncCaller) Awesomize(s string) {
	f(s)
}

func main() {
	a := FuncCaller(func(s string) {
		fmt.Println(s)
	})
	a.Awesomize("fuck")
}
```

## 嵌入

There is no subclassing in Go. Instead, there is interface and struct embedding.

```go
// ReadWriter implementations must satisfy both Reader and Writer
type ReadWriter interface {
    Reader
    Writer
}

// Server exposes all the methods that Logger has
type Server struct {
    Host string
    Port int
    *log.Logger
}

// initialize the embedded type the usual way
server := &Server{"localhost", 80, log.New(...)}

// methods implemented on the embedded struct are passed through
server.Log(...) // calls server.Logger.Log(...)

// the field name of the embedded type is its type name (in this case Logger)
var logger *log.Logger = server.Logger
```

## Errors
There is no exception handling. Functions that might produce an error just declare an additional return value of type `Error`. This is the `Error` interface:
```go
type error interface {
    Error() string
}
```

A function that might return an error:
```go
func doStuff() (int, error) {
}

func main() {
    result, err := doStuff()
    if err != nil {
        // handle error
    } else {
        // all is good, use result
    }
}
```

# Concurrency

## Goroutines
Goroutines are lightweight threads (managed by Go, not OS threads). `go f(a, b)` starts a new goroutine which runs `f` (given `f` is a function).

```go
// just a function (which can be later started as a goroutine)
func doStuff(s string) {
}

func main() {
    // using a named function in a goroutine
    go doStuff("foobar")

    // using an anonymous inner function in a goroutine
    go func (x int) {
        // function body goes here
    }(42)
}
```

## Channels
```go
ch := make(chan int) // create a channel of type int
ch <- 42             // Send a value to the channel ch.
v := <-ch            // Receive a value from ch

// Non-buffered channels block. Read blocks when no value is available, write blocks until there is a read.

// Create a buffered channel. Writing to a buffered channels does not block if less than <buffer size> unread values have been written.
ch := make(chan int, 100)

close(ch) // closes the channel (only sender should close)

// read from channel and test if it has been closed
v, ok := <-ch

// if ok is false, channel has been closed

// Read from channel until it is closed
for i := range ch {
    fmt.Println(i)
}

// select blocks on multiple channel operations, if one unblocks, the corresponding case is executed
func doStuff(channelOut, channelIn chan int) {
    select {
    case channelOut <- 42:
        fmt.Println("We could write to channelOut!")
    case x := <- channelIn:
        fmt.Println("We could read from channelIn")
    case <-time.After(time.Second * 1):
        fmt.Println("timeout")
    }
}
```

### Channel Axioms
- A send to a nil channel blocks forever

  ```go
  var c chan string
  c <- "Hello, World!"
  // fatal error: all goroutines are asleep - deadlock!
  ```
- A receive from a nil channel blocks forever

  ```go
  var c chan string
  fmt.Println(<-c)
  // fatal error: all goroutines are asleep - deadlock!
  ```
- A send to a closed channel panics

  ```go
  var c = make(chan string, 1)
  c <- "Hello, World!"
  close(c)
  c <- "Hello, Panic!"
  // panic: send on closed channel
  ```
- A receive from a closed channel returns the zero value immediately

  ```go
  var c = make(chan int, 2)
  c <- 1
  c <- 2
  close(c)
  for i := 0; i < 3; i++ {
      fmt.Printf("%d ", <-c)
  }
  // 1 2 0
  ```

## Printing

```go
fmt.Println("Hello, 你好, नमस्ते, Привет, ᎣᏏᏲ") // basic print, plus newline
p := struct { X, Y int }{ 17, 2 }
fmt.Println( "My point:", p, "x coord=", p.X ) // print structs, ints, etc
s := fmt.Sprintln( "My point:", p, "x coord=", p.X ) // print to string variable

fmt.Printf("%d hex:%x bin:%b fp:%f sci:%e",17,17,17,17.0,17.0) // c-ish format
s2 := fmt.Sprintf( "%d %f", 17, 17.0 ) // formatted print to string variable

hellomsg := `
 "Hello" in Chinese is 你好 ('Ni Hao')
 "Hello" in Hindi is नमस्ते ('Namaste')
` // multi-line string literal, using back-tick at beginning and end
```

## Reflection
### Type Switch
A type switch is like a regular switch statement, but the cases in a type switch specify types (not values), and those values are compared against the type of the value held by the given interface value.
```go
func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	do(21)
	do("hello")
	do(true)
}
```

# Snippets

## HTTP Server
```go
package main

import (
    "fmt"
    "net/http"
)

// define a type for the response
type Hello struct{}

// let that type implement the ServeHTTP method (defined in interface http.Handler)
func (h Hello) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello!")
}

func main() {
    var h Hello
    http.ListenAndServe("localhost:4000", h)
}

// Here's the method signature of http.ServeHTTP:
// type Handler interface {
//     ServeHTTP(w http.ResponseWriter, r *http.Request)
// }
```

## 函数封装结构体的初始化过程
```go
type Command struct {
	Name    string
	Var     *int
	Comment string
}

func newCommand(name string, varref *int, comment string) *Command {
	return &Command{
		Name:    name,
		Var:     varref,
		Comment: comment,
	}
}

func main() {
	var version = 1
	cmd := newCommand("version", &version, "show version")
	fmt.Printf("%T, %d", cmd, cmd)
}
```

## 使用匿名结构体

```go
func printMsgType(msg *struct {
	id   int
	data string
}) {
	fmt.Printf("%T\n", msg)
}

func main() {
	msg := &struct {
		id   int
		data string
	}{
		1024,
		"hello",
	}
	printMsgType(msg)
}
```

## playergen
```go
package main

import "fmt"

func playerGen(name string) func() (string, int) {
	hp := 150
	return func() (string, int) {
		return name, hp
	}
}
func main() {
	generator := playerGen("Victor")
	name, hp := generator()
	fmt.Println(name, hp) // Victor 15
}
```
