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
4. [Functions](#functions)
    * [Functions as values and closures](#functions-as-values-and-closures)
    * [Variadic Functions](#variadic-functions)
5. [内置数据类型](#内置数据类型)
6. [类型转换](#类型转换)
7. [Packages](#packages)
8. [Control structures](#control-structures)
    * [If](#if)
    * [Loops](#loops)
    * [Switch](#switch)
9. [数组，切片，范围](#数组-切片-范围)
    * [数组](#数组)
    * [切片](#切片)
    * [数组和切片的操作](#数组和切片的操作)
10. [映射，列表](#映射-列表)
    * [映射](#映射)
    * [sync.Map*](#sync.Map)
    * [列表*](#列表)
11. [Structs](#structs)
12. [指针](#指针)
13. [Interfaces](#interfaces)
14. [Embedding](#embedding)
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

## 感谢

多数代码摘自 [A Tour of Go](http://tour.golang.org/)，这是一个很好的 Go 入门站点。如果你是第一次接触 Go，建议你可以先去试验下。

我不打算纯翻译英文版，在读《Go 语言从入门到进阶实战》的过程中，我从书中也摘抄了一些代码补充进来。

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

// iota can be used for incrementing numbers, starting from 0
const (
    _ = iota
    a
    b
    c = 1 << iota
    d
)
    fmt.Println(a, b) // 1 2 (0 is skipped)
    fmt.Println(c, d) // 8 16 (2^3, 2^4)
```

## Functions
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

### Functions As Values And Closures
```go
func main() {
    // assign a function to a name
    add := func(a, b int) int {
        return a + b
    }
    // use the name to call the function
    fmt.Println(add(3, 4))
}

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

### Variadic Functions
```go
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
```

## 内置数据类型
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

## Control structures

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
var m map[string]int
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
// 变量名 := list.New() 或 var 变量名 list.List
l := list.New()
element := l.PushBack("first")
l.PushFront(67)
l.InsertAfter("high", element)
l.InsertBefore("noon", element)
l.Remove(element)

for i := l.Front(); i != nil; i = i.Next() {
    fmt.Println(i.Value)
}
```

## Structs

There are no classes, only structs. Structs can have methods.
```go
// A struct is a type. It's also a collection of fields

// Declaration
type Vertex struct {
    X, Y int
}

// Creating
var v = Vertex{1, 2}
var v = Vertex{X: 1, Y: 2} // Creates a struct by defining values with keys
var v = []Vertex{{1,2},{5,2},{5,5}} // Initialize a slice of structs

// Accessing members
v.X = 4

// You can declare methods on structs. The struct you want to declare the
// method on (the receiving type) comes between the the func keyword and
// the method name. The struct is copied on each method call(!)
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
**Anonymous structs:**
Cheaper and safer than using `map[string]interface{}`.
```go
point := struct {
	X, Y int
}{1, 2}
```

## 指针
分为两个核心概念：
1. 类型指针，允许对这个指针类型的数据进行修改。传递数据使用指针，而无须拷贝数据。类型指针不能进行偏移和运算。
2. 切片，由指向起始元素的原始指针、元素数量和容量组成。
```go
p := Vertex{1, 2}  // p is a Vertex
q := &p            // q is a pointer to a Vertex，`&` 操作符进行取地址操作
r := &Vertex{1, 2} // r is also a pointer to a Vertex，r 的类型是 *Vertex
value := *r        // value is `Vertex{1, 2}`, `*` 是从指针取值操作
*r = Vertex{4, 6}  // * 操作符的根本意义就是操作指针指向的变量，在右侧是取变量的值，在左侧就是将值设置给指向的变量

// Vertex 的指针类型是 *Vertex
var s *Vertex = new(Vertex) // new(类型) 是创建指针的另外一种方法，指针指向的值为默认值
```

## Interfaces
```go
// interface declaration
type Awesomizer interface {
    Awesomize() string
}

// types do *not* declare to implement interfaces
type Foo struct {}

// instead, types implicitly satisfy an interface if they implement all required methods
func (foo Foo) Awesomize() string {
    return "Awesome!"
}
```

## Embedding

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


