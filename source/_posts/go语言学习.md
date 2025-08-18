---
title: go语言学习
date: 2025-08-13 11:08:23
categories:
  - "后端"       # 一级分类
  - "go"
tags:
---
阅读[《go语言圣经》](https://books.studygolang.com/gopl-zh/) 后的简单整理总结

## 程序结构

### 命名
命名规则：以字母或者下划线开头，后面跟任意数量的字母、下划线或者数字，大小写敏感。
关键字有25个，不能用于自定义名称
``` go
break      default       func     interface   select
case       defer         go       map         struct
chan       else          goto     package     switch
const      fallthrough   if       range       type
continue   for           import   return      var
```

预定义名称，主要对应内建的常量、类型和函数等。(命名时虽然可以使用，但是最好不要使用，所以出现命名冲突)
``` go
内建常量: true false iota nil

内建类型: int int8 int16 int32 int64
         uint uint8 uint16 uint32 uint64 uintptr
         loat32 float64 complex128 complex64
         bool byte rune string error

内建函数: make len cap new append copy close delete
         complex real imag
         panic recover
```

### 声明
声明语句：
``` go
var   变量
const 常量
func  函数
type  类型
```
go源文件以包声明语句开始，说明源文件是属于那一个包。后面上 import 导入语句，然后上包一级的类型、变量、常量、函数的声明语句，包一级的各种声明语句没有顺序要求，函数内部的名字必须先声明后使用。

### 变量
var 声明语句可以创建一个特定类型的变量，并且设置初始变量值，语法如下：
``` go
var 变量名 数据类型 = 初始值
```
其中 **类型** 和 **= 初始值** 是可选的。省略后面的 **类型**，会根据初始化表达式推导变量类型。省略 **= 初始值** ，变量的值为零值，数值类型变量零值为 0， 布尔类型变量零值为 false，字符串类型变量零值为 ""（空字符串），接口或引用类型（包括slice、指针、map、chan和函数）变量零值为 nil。数组或结构体等聚合类型对应的零值是每个元素或字段都是对应该类型的零值。

可以在声明语句中同时声明一组变量，省略变量类型可以声明多个不同类型的变量（变量类型由初始化表达式推导出来）：
``` go
var i, j, k int                 // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
```

#### 简短变量声明
在声明语句中，可以使用 “名字 := 表达式” 声明和初始化局部变量，变量类型由初始化表达式推导出来：
``` go
name := "Tom"
t := 0.1
j, i := 1, "2"
```
简单声明语句中必须要至少声明一个新的变量，下面的代码将编译失败：
``` go
f, err := os.Open(infile)
// ...
f, err := os.Create(outfile) // compile error: no new variables
```
解决办法是把第二个改为赋值语句。

#### 指针
在 Go 中，指针是一个内存地址。指针变量的值是内存地址，指针变量的类型是内存地址的类型的指针，一个指针对应变量在内存中的存储位置。 
`*` 和 `&` 是指针相关的核心运算符，分别表示 **解引用** 和 **取地址**。
**核心用途**： 函数间共享数据；动态内存分配（new）；构建链表、树等数据结构。

---
1. `&` （取地址运算符） 
  **作用**： 获取变量的内存地址，生成指向该变量的指针。 
  **语法**： `ptr := &var` `var`: 任意类型的变量。 `ptr`：存储变量地址的指针，类型为 `*T`（`T` 是变量类型）。 
  **示例**： 
  ``` go
  x := 42
  ptr := &x // ptr 类型为 *int，存储 x 的内存地址
  fmt.Printf("ptr 的值（地址）: %p\n", ptr) // 输出类似 0xc0000a2000
  ```
  **关键点**：只能获取 **可寻址值** 的地址（如变量、结构体字段、数组元素，但不能是字面量或临时结果）。 
  ``` go
  &42          // 错误：无法获取字面量的地址
  &getValue()  // 错误：无法获取函数返回值的地址
  ```

2. `*` （解引用运算符） 
  **作用**： 通过指针访问或修改其指向的内存中的值。 
  **语法**： 
  ``` go
  value := *pointer  // 读取指针指向的值
  *pointer = newValue // 修改指针指向的值
  ```
 **示例**： 
  ``` go
  x := 42
  ptr := &x
  fmt.Println(*ptr) // 输出 42（读取指针指向的值）

  *ptr = 100        // 通过指针修改 x 的值
  fmt.Println(x)    // 输出 100
  ```
  **关键点**： 解引用 **空指针（`nil`）** 会导致panic：
  ``` go
  var ptr *int
  fmt.Println(*ptr)  // panic: runtime error: invalid memory address or nil pointer dereference
  ```

---
**指针和引用类型的区别**：

| 特性     | 指针（*T）          | 引用类型（如 slice、map） |
| -------- | ------------------- | ------------------------- |
| 底层存储 | 直接存储地址        | 包含底层数据的指针+元信息 |
| 显式操作 | 需用 * 和 &         | 自动解引用，无需显式操作  |
| 安全性   | 可能为空指针（nil） | 运行时检查，避免空指针    |

#### new函数
另一个创建变量的方法是调用内建的new函数。表达式new(T)将创建一个T类型的匿名变量，初始化为T类型的零值，然后返回变量地址，返回的指针类型为*T。
``` go
p := new(int)   // p, *int 类型, 指向匿名的 int 变量
fmt.Println(*p, p) // "0"  "0xc000010070"
*p = 2          // 设置 int 匿名变量的值为 2
fmt.Println(*p) // "2"
```
new函数创建变量与普通变量声明语句创建变量没有区别，new函数类似一个语法糖，如下两个函数有相同行为：
``` go
func newInt() *int {
  return new(int)
}

func newInt() *int {
  var dummy int
  return &dummy
}
```

### 类型
变量或表达式的类型定义了对于存储值的属性特征，例如数值在内存的存储大小（或者是元素的bit个数），它们在内部是如何表达的，是否支持一些操作符，以及它们自己关联的方法集等。

一个类型声明语句创建了一个新的类型名称，和现有类型具有相同的底层结构。新命名的类型提供了一个方法，用来分隔不同概念的类型，这样即使它们底层类型相同也是不兼容的。
``` go
type 类型名称 底层类型
```
类型声明语句一般出现在包一级，因此如果类型名称首字母大写的话，则在包外部也可以使用。
> 注：由于汉字不区分大小写，所以汉字开头的名称没有导出

### 包和文件
Go语言的包与其他语言的库或模块的概率类似，目的都是为了支持模块化、封装、单独编译和代码重用。一个包的源代码保存在一个或多个以.go为文件后缀名的源文件中，通常一个包所在目录路径的后缀是包的导入路径。

每个包都对应一个独立的名字空间。例如，在image包中的Decode函数和在unicode/utf16包中的 Decode函数是不同的。要在外部引用该函数，必须显式使用image.Decode或utf16.Decode形式访问。

#### 导入包
每个包都有一个全局唯一的导入路径，导入语句中类似"golang.org/x/net/html"的字符串对应包的导入路径。Go语言的规范并没有定义这些字符串的具体含义或包来自哪里，它们是由构建工具来解释的。当使用Go语言自带的go工具箱时，一个导入路径代表一个目录中的一个或多个Go源文件。

除了包的导入路径，每个包还有一个包名，包名一般是短小的名字（并不要求包名是唯一的），包名在包的声明处指定。按照惯例，一个包的名字和包的导入路径的最后一个字段
相同，导入语句将导入的包绑定到一个短小的名字，然后通过该短小的名字就可以引用包中导出的全部内容。

#### 包的初始化
包的初始化首先是解决包级变量的依赖顺序，然后按照包级变量声明出现的顺序依次初始化：
``` go
var a = b + c // a 第三个初始化, 为 3
var b = f()   // b 第二个初始化, 为 2, 通过调用 f (依赖c)
var c = 1     // c 第一个初始化, 为 1

func f() int { return c + 1 }
```
如果包中含有多个.go源文件，它们将按照发给编译器的顺序进行初始化，Go语言的构建工具首先会将.go文件根据文件名排序，然后依次调用编译器编译。

对于在包级别声明的变量，如果有初始化表达式则用表达式初始化，还有一些没有初始化表达式的，例如某些表格数据初始化并不是一个简单的赋值过程。在这种情况下，我们可以用一个特殊的init初始化函数来简化初始化工作。每个文件都可以包含多个init初始化函数
``` go
func init() { /* ... */ }
```
这样的init初始化函数除了不能被调用或引用外，其他行为和普通函数类似。在每个文件中的init初始化函数，在程序开始执行时按照它们声明的顺序被自动调用。
> 注：也可以通过将初始化逻辑包装为一个匿名函数处理。
> var pc = func() (类型) {/** 初始化逻辑 **/}()

### 作用域
声明语句的作用域指源代码中可以有效使用这个名字的范围，声明语句对应的词法域决定了作用域范围的大小。

1. 包级作用域：包级作用域的变量，可以在整个包中访问。
2. 函数级作用域：函数级作用域的变量，只能在函数内部访问。
3. 块级作用域：块级作用域的变量，只能在当前块中访问（即用花括号 {} 括起来的部分）。

一个程序可以包含多个同名的声明，只要它们在不同的作用域就可以。

#### if else 语句 
```go
if condition {
  // 执行语句
} else {
  // 执行语句
}
```
if语句的else部分是可选的，在if的条件为false时执行。

#### 循环语句
Go语言只有for循环这一种循环语句。for循环有多种形式
```go
for initialization; condition; post {
  // 执行语句
}
```
for循环三个部分不需括号包围。大括号强制要求，左大括号必须和post语句在同一行。

initialization语句是可选的，在循环开始前执行。initalization如果存在，必须是一条简单语句（simple statement），即，短变量声明、自增语句、赋值语句或函数调用。condition是一个布尔表达式（boolean expression），其值在每次循环迭代开始时计算。如果为true则执行循环体语句。post语句在循环体执行结束后执行，之后再次对condition求值。condition值为false时，循环结束。
```go
// 传统的 while 循环
for condition {
  // ...
}

// 无条件无限循环，使用 break 或 return 退出
for {
  // ...
}

// 迭代数组，类似于 JavaScript 中的 forEach
for _, arg := range []int{1, 2, 3} {
  // ...
}

// 循环标签
loop:
	for {
		select {
		case size, ok := <-fileSizes:
			if !ok {
				break loop // 这里break跳出标签loop，没有使用会只是内存循环的break
			}
		case <-tick:
		}
	}
```

#### switch 语句
`switch` 语句 是一种多分支条件控制结构，比传统的 if-else 链更简洁清晰。它支持多种灵活用法，包括值匹配、类型判断和省略条件表达式。
```go
switch 表达式 {
case 值1:
    // 匹配值1时执行
case 值2, 值3: // 多个值用逗号分隔
    // 匹配值2或值3时执行
default:
    // 默认分支（可选）
}

// 省略表达式（相当于 if-else）直接在每个 case 中写条件表达式：
score := 85
switch {
case score >= 90:
    fmt.Println("A")
case score >= 80:
    fmt.Println("B") // 输出
case score >= 60:
    fmt.Println("C")
}
```
Go 的 switch 默认执行匹配的 case 后自动退出，不会继续执行后续分支。如需继续执行下一分支，使用 fallthrough：
```go
n := 2
switch n {
case 1:
    fmt.Println("n is 1")
case 2:
    fmt.Println("n is 2") // 输出
    fallthrough           // 强制执行下一个 case
case 3:
    fmt.Println("n is 3") // 也会输出
}
```

#### select 语句
`select` 语句是专门为 通道（Channel） 设计的一种多路复用控制结构，类似于 `switch`，但它的每个 `case` 必须是一个通道操作（发送或接收）。select 会监听多个通道的操作，当其中某个 `case` 准备就绪（通道可读或可写）时，就会执行对应的分支。如果多个 `case` 同时就绪，`select` 会随机选择一个执行。 
`select` 是 Go 并发编程的核心工具之一，用于协调多个通道操作。适用于事件循环、任务调度、超时处理等场景。
```go
select {
case <-ch1:
    // 如果 ch1 可读，执行此分支
case ch2 <- value:
    // 如果 ch2 可写，执行此分支
default:
    // 如果没有任何 case 就绪，执行默认分支（可选）
}
```

##### 核心特性
**非阻塞检查**: 如果没有 `default` 分支，`select` 会阻塞，直到至少一个 `case` 就绪；如果有 `default`，则会直接执行它（实现非阻塞操作）。 
**随机选择**: 如果多个 `case` 同时就绪，`select` 会随机选择一个执行（避免饥饿问题）。 
**超时控制**: 结合 `time.After` 可以实现超时机制:
```go
select {
case <-ch:
    fmt.Println("收到数据")
case <-time.After(2 * time.Second):
    fmt.Println("超时！")
}
```
**永久阻塞**: 空的 `select`（`select {}`）会永久阻塞，常用于阻止 `main` 函数退出。

## 基础数据类型
虽然从底层而言，所有的数据都是由比特组成，但计算机一般操作的是固定大小的数，如整数、浮点数、比特数组、内存地址等。进一步将这些数组织在一起，就可表达更多的对象，例如数据包、像素点、诗歌，甚至其他任何对象。Go语言提供了丰富的数据组织形式，这依赖于Go语言内置的数据类型。这些内置的数据类型，兼顾了硬件的特性和表达复杂数据结构的便捷性。

Go语言将数据类型分为四类：基础类型、复合类型、引用类型和接口类型。本章介绍基础类型，包括：数字、字符串和布尔型。复合数据类型——数组和结构体——是通过组合简单类型，来表达更加复杂的数据结构。引用类型包括指针、切片、字典、函数、通道，虽然数据种类很多，但它们都是对程序中一个变量或状态的间接引用。这意味着对任一引用类型数据的修改都会影响所有该引用的拷贝。

### 整数型
Go语言的数值型包括整数、浮点数和复数。每种数值类型都决定了对应的大小范围和是否支持正负符号。

#### 整数类型

Go语言提供了有符号和无符号类型的整数运算。有int8、int16、int32和int64四种不同大小的有符号整数类型，分别对应8、16、32、64bit大小的有符号整数，与此对应的是uint8、uint16、uint32和uint64四种无符号整数类型。

有符号整数：
| 类型  | 位数（Bits） | 取值范围（最小值 ~ 最大值）                            | 示例                    |
| ----- | ------------ | ------------------------------------------------------ | ----------------------- |
| int   | 32 或 64     | 取决于平台（同 int32 或 int64）                        | var x int = 42          |
| int8  | 8            | -128 ~ 127                                             | var x int8 = -100       |
| int16 | 16           | -32,768 ~ 32,767                                       | var x int16 = 30000     |
| int32 | 32           | -2,147,483,648 ~ 2,147,483,647                         | var x int32 = 1_000_000 |
| int64 | 64           | -9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807 | var x int64 = 1e18      |

无符号整数：
| 类型   | 位数（Bits） | 取值范围（0 ~ 最大值）            | 示例                         |
| ------ | ------------ | --------------------------------- | ---------------------------- |
| uint   | 32 或 64     | 取决于平台（同 uint32 或 uint64） | var x uint = 42              |
| uint8  | 8            | 0 ~ 255                           | var x uint8 = 200            |
| uint16 | 16           | 0 ~ 65,535                        | var x uint16 = 60000         |
| uint32 | 32           | 0 ~ 4,294,967,295                 | var x uint32 = 3_000_000_000 |
| uint64 | 64           | 0 ~ 18,446,744,073,709,551,615    | var x uint64 = 1e19          |

这里还有两种一般对应特定CPU平台机器字大小的有符号和无符号整数int和uint；其中int是应用最广泛的数值类型。这两种类型都有同样的大小，32或64bit，但是我们不能对此做任何的假设；因为不同的编译器即使在相同的硬件平台上可能产生不同的大小。

Unicode字符rune类型是和int32等价的类型，通常用于表示一个Unicode码点。这两个名称可以互换使用。同样byte也是uint8类型的等价类型，byte类型一般用于强调数值是一个原始的数据而不是一个小的整数。

最后，还有一种无符号的整数类型uintptr，没有指定具体的bit大小但是足以容纳指针。uintptr类型只有在底层编程时才需要，特别是Go语言和C语言函数库或操作系统接口相交互的地方。

| 类型    | 位数（Bits） | 说明                                     | 示例                                        |
| ------- | ------------ | ---------------------------------------- | ------------------------------------------- |
| uintptr | 32 或 64     | 足够存储指针地址的整数（用于 unsafe 包） | var p uintptr = uintptr(unsafe.Pointer(&x)) |
| byte    | 8            | uint8 的别名，表示字节                   | var b byte = 'A'                            |
| rune    | 32           | int32 的别名，表示 Unicode 码点          | var r rune = '中'                           |

不管它们的具体大小，int、uint和uintptr是不同类型的兄弟类型。其中int和int32也是不同的类型，即使int的大小也是32bit，在需要将int当作int32类型的地方需要一个显式的类型转换操作，反之亦然。

其中有符号整数采用2的补码形式表示，也就是最高bit位用来表示符号位，一个n-bit的有符号数的值域是从$-2^{n-1}$到$2^{n-1}-1$。无符号整数的所有bit位都用于表示非负数，值域是0到$2^n-1$。例如，int8类型整数的值域是从-128到127，而uint8类型整数的值域是从0到255。

#### 运算符

下面是Go语言中关于算术运算、逻辑运算和比较运算的二元运算符，它们按照优先级递减的顺序排列：
``` go
*      /      %      << 左移     >> 右移     & 位运算 AND      &^ 位清空（AND NOT）
+      -      | 位运算 OR      ^ 位运算 XOR
==     !=     <      <=       >      >=
&&
||
```
如何选择整数类型：
| 场景                   | 推荐类型    | 原因                                     |
| ---------------------- | ----------- | ---------------------------------------- |
| 通用整数计算           | int         | 平台最优性能，适合数组索引、循环计数器等 |
| 需要明确位宽           | int32/int64 | 跨平台兼容（如网络协议、文件格式）       |
| 非负数（如大小、索引） | uint        | 避免负数，扩大正数范围                   |
| 二进制数据（字节）     | byte        | 语义明确，等同于 uint8                   |
| Unicode 字符           | rune        | 支持 4 字节 UTF-8 编码（等同于 int32）   |

> 默认选择 int/uint，除非需要明确位宽或特殊语义。

### 浮点数
Go语言提供了两种精度的浮点数，float32和float64。它们的算术规范由IEEE754浮点数国际标准定义，该浮点数规范被所有现代的CPU支持。
#### 浮点数类型
| 类型    | 位数（Bits） | 取值范围（近似）      | 精度（十进制位数） | 示例                     |
| ------- | ------------ | --------------------- | ------------------ | ------------------------ |
| float32 | 32           | ±1.18e-38 ~ ±3.4e38   | 6~7                | var x float32 = 3.14     |
| float64 | 64           | ±2.23e-308 ~ ±1.8e308 | 15~16              | var y float64 = 6.022e23 |

**关键点** 
`float64` 是默认选择：更高的精度和范围，Go 的数学函数（如 `math.Sqrt`）通常返回 `float64`。 
`float32` 适用场景：内存敏感型应用（如大规模浮点数组），与外部系统交互时需要明确 32 位浮点数（如某些 GPU 计算）。
小数点前面或后面的数字都可能被省略（例如.707或1.）。很小或很大的数最好用科学计数法书写，通过e或E来指定指数部分：
``` go
const Avogadro = 6.02214129e23  // 阿伏伽德罗常数
const Planck   = 6.62606957e-34 // 普朗克常数
```
#### 特殊值
浮点数包含以下 IEEE 754 定义的特殊值：
| 值          | 表示 | 说明                              |
| ----------- | ---- | --------------------------------- |
| 正无穷大    | +Inf | 超出最大可表示值（如 1.0 / 0.0）  |
| 负无穷大    | -Inf | 超出最小可表示值（如 -1.0 / 0.0） |
| 非数（NaN） | NaN  | 无效运算结果（如 0.0 / 0.0）      |

**检查特殊值**：
``` go
import "math"

x := math.Inf(1)  // +Inf
y := math.NaN()   // NaN

fmt.Println(math.IsInf(x, 1)) // true
fmt.Println(math.IsNaN(y))    // true
```

如何选择浮点类型： 
| 场景                     | 推荐类型 | 原因                                             |
| ------------------------ | -------- | ------------------------------------------------ |
| 通用计算                 | float64  | 更高的精度和范围，避免累积误差                   |
| 内存敏感型数据（如数组） | float32  | 减少内存占用（但需注意精度损失）                 |
| 科学计算或金融领域       | float64  | 需要高精度运算（或使用 math/big 包处理任意精度） |
> 注意精度问题：避免直接比较浮点数，使用误差范围判断。
> 特殊值：Inf 和 NaN 需特殊处理。

### 复数
复数（Complex Number）是复数系统（复数域）中的元素，它由实部（Real Part）和虚部（Imaginary Part）组成。形式为 `z = a + bi`，其中 `a` 为实部，`b` 为虚部，`i`为虚数单位，满足`i² = -1`。 

Go语言中提高两种精度的复数类型，complex64和complex128，分别对应float32和float64两种浮点数精度。内置的complex函数用于构建复数，内建的real和imag函数分别返回复数的实部和虚部：
``` go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
```

### 布尔型
布尔型变量的值为 true 或 false。if 和 for 循环的判断条件为布尔型。布尔型不会隐式转换为数字。

### 字符串
字符串（string） 是一个不可变的（immutable）字节序列，可以包含任意的数据，包括byte值0，但通常用于表示文本数据。它采用 UTF-8 编码，支持 Unicode 字符。

#### 基本特性
**(1) 不可变性（Immutable）**
- 字符串一旦创建，其内容 不可修改。
- 任何修改操作（如拼接、替换）都会生成新的字符串。

**(2) UTF-8 编码**
- Go 的字符串默认使用 UTF-8 编码，支持全球所有语言的字符（包括中文、Emoji 等）。
- 每个字符可能占用 1~4 个字节（ASCII 字符占 1 字节，中文占 3 字节）。

**(3) 底层结构**
- 字符串的底层是一个 只读的字节切片（[]byte），可以通过类型转换互转。
- 运行时表示为：
``` go
type string struct {
  ptr *byte // 指向底层字节数组的指针
  len int   // 字符串长度（字节数）
}
```

#### 字符串的声明与初始化
**基本声明**：
``` go
var s1 string           // 默认值为空字符串 ""
s2 := "Hello, 世界"     // 双引号包裹（支持转义字符）
s3 := `Raw\nString`     // 反引号包裹（原始字符串，不转义）
```

**转义字符**：
| 转义序列 | 含义         | 示例               |
| -------- | ------------ | ------------------ |
| \a       | 响铃         | "Line1\aLine2"     |
| \b       | 退格         |                    |
| \f       | 换页         | "Line1\fLine2"     |
| \n       | 换行         | "Line1\nLine2"     |
| \r       | 回车         | "Line1\rLine2"     |
| \t       | 制表符       | "Name:\tAlice"     |
| \v       | 垂直制表符   | "Name:\vAlice"     |
| \\       | 反斜杠       | "C:\\path"         |
| \'       | 单引号       | 'She said, \'Hi\'' |
| \"       | 双引号       | "She said, \"Hi\"" |
| \uXXXX   | Unicode 码点 | "\u4e2d" 表示 "中" |

#### 字符串操作
**拼接**
``` go
s1 := "Hello"
s2 := "World"
result := s1 + ", " + s2  // "Hello, World"
```
**长度**
- 字节长度：`len(s)` 返回字符串的字节数。
- 字符长度：需先转换为 `[]rune`：
``` go
s := "Hello, 世界"
byteLen := len(s)       // 13（ASCII 字符 + 中文字符的字节数）
charLen := len([]rune(s)) // 8（实际字符数）
```
**索引访问**
- 通过索引获取的是**字节（byte）**，而非字符：
``` go
s := "A界"
fmt.Println(s[0]) // 65（'A' 的 ASCII 码）
fmt.Println(s[1]) // 231（中文字符的第一个字节）
```
**子串切片**
- 通过索引获取的字符串切片，**包含索引指定的字符**：
``` go
s := "Hello, 世界"
sub := s[7:10] // "世"（切片基于字节位置）
```

#### 字符串与其他类型的转换
**字符串 ↔ 字节切片（`[]byte`）**
``` go
// string → []byte
bytes := []byte("Hello") // [72 101 108 108 111]

// []byte → string
s := string([]byte{72, 101, 108, 108, 111}) // "Hello"
```
** 字符串 ↔ Unicode 码点（`[]rune`）**
``` go
// string → []rune
runes := []rune("世界") // [19990 30028]

// []rune → string
s := string([]rune{19990, 30028}) // "世界"

// 使用`range`遍历字符串
str := "Go语言"
// 方式1：range 自动按 rune 迭代
for i, r := range str {  // r 的类型是 rune
    fmt.Printf("%d: %c (Unicode: %U)\n", i, r, r)
}
// 输出：
// 0: G (U+0047)
// 1: o (U+006F)
// 2: 语 (U+8BED)
// 5: 言 (U+8A00)  // 注意：i 跳过了 3 和 4（因为"语"占 3 字节）

// 方式2：显式转换为 []rune
for i, r := range []rune(str) {
    fmt.Printf("%d: %c\n", i, r)  // i 连续递增
}
```
**字符串 ↔ 数字**
``` go
x := 123
y := fmt.Sprintf("%d", x)
fmt.Println(y, strconv.Itoa(x)) // "123 123"

x, err := strconv.Atoi("123")             // x is an int
y, err := strconv.ParseInt("123", 10, 64) // base 10, up to 64 bits
```

#### 字符串高效处理
**避免频繁拼接**
``` go
var builder strings.Builder
builder.WriteString("Hello")
builder.WriteString(", ")
builder.WriteString("World")
result := builder.String() // "Hello, World"
```
**常用标准库**
- `strings`：字符串操作（分割、替换、查找等）。
``` go
import "strings"
strings.Contains("hello", "ell") // true
strings.Split("a,b,c", ",")      // ["a", "b", "c"]
```
- `strconv`：字符串与其他类型的转换。
``` go
num, _ := strconv.Atoi("42")    // string → int
s := strconv.Itoa(42)          // int → string
```
> **不可变**：字符串内容无法直接修改，操作会生成新字符串。
> **UTF-8 编码**：支持多语言文本，但需注意字节与字符的区别。

### 常量
常量表达式的值在编译期计算，而不是在运行期。每种常量的潜在类型都是基础类型：boolean、string或数字。

一个常量的声明语句定义了常量的名字，和变量的声明语法类似，常量的值不可修改，这样可以防止在运行期被意外或恶意的修改。例如，常量比变量更适合用于表达像π之类的数学常数，因为它们的值不会发生变化：`const pi = 3.14159`

所有常量的运算都可以在编译期完成，这样可以减少运行时的工作，也方便其他编译优化。当操作数是常量时，一些运行时的错误也可以在编译时被发现，例如整数除零、字符串索引越界、任何导致无效浮点数的操作等。

#### iota 常量生成器
常量声明可以使用iota常量生成器初始化，它用于生成一组以相似规则初始化的常量，但是不用每行都写一遍初始化表达式。在一个const声明语句中，在第一个声明的常量所在的行，iota将会被置为0，然后在每一个有常量声明的行加一。
``` go
const (
  a = iota
  b
  c
  d
)
fmt.Println(a, b, c, d)  // 0 1 2 3
```
也可以在复杂的常量表达式里使用iota：
``` go
const (
	a = 1 << iota
	b
	c
	d
)
fmt.Printf("%b %#[1]v\n", a) // 1  1
fmt.Printf("%b %#[1]v\n", b) // 10  2
fmt.Printf("%b %#[1]v\n", c) // 100  4
fmt.Printf("%b %#[1]v\n", d) // 1000  8
```

#### 无类型常量（Untyped Constants）
无类型常量是 Go 语言中一种特殊的常量形式，它具有比普通类型化常量更灵活的特性。

无类型常量是指在声明时没有显式指定类型的常量。例如：
``` go
const pi = 3.14159      // 无类型浮点常量
const hello = "Hello"   // 无类型字符串常量
const truth = true      // 无类型布尔常量
```
与之相对的是有类型常量：
``` go
const pi float64 = 3.14159  // 有类型常量（float64）
```

##### 无类型常量的特性
延迟类型确定：无类型常量在声明时不绑定具体类型，直到在表达式中使用时才根据上下文确定类型。这使得它们可以更灵活地参与不同类型的运算。
``` go
const a = 10  // 无类型整型常量

var i int = a       // 作为int使用
var f float64 = a   // 作为float64使用
var b byte = a      // 作为byte使用
```
更高的精度： 无类型数值常量在 Go 中具有比普通变量更高的精度，可以表示非常大的数值而不会溢出。
``` go
const huge = 1 << 100  // 合法，无类型常量可以表示非常大的数
var h int = huge >> 96  // 合法，右移后可以赋给int
```
默认类型： 虽然无类型常量没有固定类型，但它们有默认类型，当需要明确类型时会使用默认类型：
- 整型常量：`int`
- 浮点常量：`float64`
- 复数常量：`complex128`
- 字符常量：`rune` (int32的别名)
- 字符串常量：`string`
- 布尔常量：`bool`
``` go
const c = 'a'      // 无类型rune常量
var r rune = c     // 作为rune使用
var i int = c      // 也可以作为int使用
```

##### 无类型常量的底层实现
Go 语言的无类型常量是通过以下方式实现的：
- 编译期处理：常量表达式在编译期完全计算
- 任意精度：数值常量使用数学上的无限精度表示
- 延迟绑定：类型信息直到使用时才确定

这使得 Go 可以处理非常大的常量值, 如：
``` go
const (
  Gigabyte = 1 << (10 * 3)
  Terabyte = 1 << (10 * 4)
)
```

> 优先使用无类型常量：除非有明确需要指定类型
> 注意默认类型：了解常量在不同上下文中的默认类型
> 大数处理：利用无类型常量的高精度特性处理大数
> 类型安全：在需要明确类型的地方进行显式转换

## 复合数据类型
数组是由同构的元素组成——每个数组元素都是完全相同的类型——结构体则是由异构的元素组成的。数组和结构体都是有固定内存大小的数据结构。相比之下，slice和map则是动态的数据结构，它们将根据需要动态增长。

### 数组
数组由一个固定长度的的特定元素组成的序列，一个数组可以有零个或多个元素组成，长度是固定的。

``` go
var a [3]int  // 定义一个长度为3的int数组
a[0] = 1      // 给数组的索引0赋值
fmt.Println(a[0], a[1])  // 打印数组的索引0和索引1的值: 1  0

var b [3]int = [3]int{1, 2, 3}  // 定义并使用数组字面值语法来初始化一个长度为3的int数组
var c [3]int = [3]int{1, 2}
fmt.Println(c[2]) // "0"
```
在数组字面值中，如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算。因此，可以简化为：
```go
var b = [...]int{1, 2, 3}
fmt.Printf("%T\n", q) // "[3]int"
```
数组的长度是数组类型的一个组成部分，因此[3]int和[4]int是两种不同的数组类型。数组的长度必须是常量表达式，因为数组的长度需要在编译阶段确定。
```go
q := [3]int{1, 2, 3}
q = [4]int{1, 2, 3, 4} // cannot use [4]int{…} (value of type [4]int) as [3]int value in assignment
```
数组、slice、map和结构体字面值的写法都很相似。上面的形式是直接提供顺序初始化值序列，但是也可以指定一个索引和对应值列表的方式初始化。
```go
type Currency int

const (
  USD Currency = iota // 美元
  EUR                 // 欧元
  GBP                 // 英镑
  RMB                 // 人民币
)
symbol := [...]string{USD: "$", EUR: "€", GBP: "￡", RMB: "￥"}
fmt.Println(RMB, symbol[RMB]) // "3 ￥"
```
在这种形式的数组字面值形式中，初始化索引的顺序是无关紧要的，而且没用到的索引可以省略，和前面提到的规则一样，未指定初始值的元素将用零值初始化。
```go
r := [...]int{99: -1} // 定义了一个含有100个元素的数组r，最后一个元素被初始化为-1，其它元素都是用0初始化。
```
数组类型可以相互比较，只有当两个数组的所有元素都是相等的时候数组才是相等的。
```go
a := [2]int{1, 2}
b := [...]int{1, 2}
c := [2]int{1, 3}
fmt.Println(a == b, a == c, b == c) // "true false false"
d := [3]int{1, 2}
fmt.Println(a == d) // invalid operation: a == d (mismatched types [2]int and [3]int)
```

### Slice（切片）
Slice（切片）代表变长的序列，序列中每个元素都有相同的类型。一个slice类型一般写作[]T，其中T代表slice中元素的类型；slice的语法和数组很像，只是没有固定长度而已。

数组和slice之间有着紧密的联系。一个slice是一个轻量级的数据结构，提供了访问数组子序列（或者全部）元素的功能，而且slice的底层确实引用一个数组对象。一个slice由三个部分构成：指针、长度和容量。指针指向第一个slice元素对应的底层数组元素的地址，要注意的是slice的第一个元素并不一定就是数组的第一个元素。长度对应slice中元素的数目；长度不能超过容量，容量一般是从slice的开始位置到底层数据的结尾位置。内置的len和cap函数分别返回slice的长度和容量。

如果切片操作超出cap(s)的上限会导致一个panic异常，但是超出len(s)则是意味着扩展了slice，因为新slice的长度会变大。

调用函数传参切片时，在函数里修改切片会直接修改底层数组，影响所有共享该数组的切片。与值类型的区别：
| 操作             | int（值类型）    | []int（引用类型）    |
| ---------------- | ---------------- | -------------------- |
| 传递方式         | 值传递（拷贝值） | 值传递（拷贝切片头） |
| 是否影响外部变量 | 否               | 是（共享底层数组）   |
| 底层行为         | 完全独立         | 共享数据             |

但是如果切片状函数里发生了扩容，（如 append 超出 cap），会分配新数组，此时修改不会影响原切片：
```go
v := []int{1, 2, 3}
setArr(v)
fmt.Println(v) // [1 2 3]
func setArr(v []int) {
  v = append(v, 4) // 扩容后，v 指向新数组
  v[0] = 2         // 不影响 main.x
}
```

#### append函数
内置的append函数用于向slice追加元素：
```go
var runes []rune
for _, r := range "Hello, 世界" {
  runes = append(runes, r)
}
fmt.Printf("%q\n", runes) // "['H' 'e' 'l' 'l' 'o' ',' ' ' '世' '界']"
```
每次调用appendInt函数，必须先检测slice底层数组是否有足够的容量来保存新添加的元素。如果有足够空间的话，直接扩展slice（依然在原有的底层数组之上），将新添加的y元素复制到新扩展的空间，并返回slice。因此，输入的x和输出的z共享相同的底层数组。

如果没有足够的增长空间的话，appendInt函数则会先分配一个足够大的slice用于保存新的结果，先将输入的x复制到新的空间，然后添加y元素。结果z和输入的x引用的将是不同的底层数组。

#### Slice内存技巧
一个slice可以用来模拟一个stack。最初给定的空slice对应一个空的stack，然后可以使用append函数将新的值压入stack：
```go
stack = append(stack, v) // push v
```
stack的顶部位置对应slice的最后一个元素：
```go
top := stack[len(stack)-1] // top of stack
```
通过收缩stack可以弹出栈顶的元素
```go
stack = stack[:len(stack)-1] // pop
```
要删除slice中间的某个元素并保存原有的元素顺序，可以通过内置的copy函数将后面的子slice向前依次移动一位完成：
```go
func remove(slice []int, i int) []int {
  copy(slice[i:], slice[i+1:])
  return slice[:len(slice)-1]
}

func main() {
  s := []int{5, 6, 7, 8, 9}
  fmt.Println(remove(s, 2)) // "[5 6 8 9]"
}
```
如果删除元素后不用保持原来顺序的话，我们可以简单的用最后一个元素覆盖被删除的元素：
```go
func remove(slice []int, i int) []int {
  slice[i] = slice[len(slice)-1]
  return slice[:len(slice)-1]
}

func main() {
  s := []int{5, 6, 7, 8, 9}
  fmt.Println(remove(s, 2)) // "[5 6 9 8]
}
```

### Map
哈希表是一种巧妙并且实用的数据结构。它是一个无序的key/value对的集合，其中所有的key都是不同的，然后通过给定的key可以在常数时间复杂度内检索、更新或删除对应的value。

map类型写为`map[K]V`，其中K和V分别对应key和value。内置的make函数可以创建一个map：
```go
ages := make(map[string]int) // 创建一个字符串到整数的映射
```
也可以使用map字面值语法创建map：
```go
ages := map[string]int{
  "alice":   31,
  "charlie": 34,
}

// 等价于
ages := make(map[string]int)
ages["alice"] = 31
ages["charlie"] = 34

names := map[int]string{} // 创建一个空的map表达式
```
Map中的元素可以使用内置的delete函数删除
```go
delete(ages, "alice")
```
所有对Map元素操作都是安全的，即使这些元素不在 Map中，查找失败会返回value类型对应的零值。
```go
ages["bob"]  // 返回零值： 0
ages["bob"] += 1 // ages["bob"] = ages["bob"] + 1 结果是1
```
判断mapl中是否存在某个元素，
```go
ge, ok := ages["bob"]
if !ok { /* "bob" is not a key in this map; age == 0. */ }

// 等价于
if age, ok := ages["bob"]; !ok { /* ... */ }
```
在这种场景下，map的下标语法将产生两个值；第二个是一个布尔值，用于报告元素是否真的存在。布尔变量一般命名为ok，特别适合马上用于if条件判断部分。

### 结构体
结构体是一种聚合的数据类型，是由零个或多个任意类型的值聚合成的实体。每个值称为结构体的成员。用结构体的经典案例是处理公司的员工信息，每个员工信息包含一个唯一的员工编号、员工的名字、家庭住址、出生日期、工作岗位、薪资、上级领导等等。所有的这些信息都需要绑定到一个实体中，可以作为一个整体单元被复制，作为函数的参数或返回值，或者是被存储到数组中，等等。

下面两个语句声明了一个叫Employee的命名的结构体类型，并且声明了一个Employee类型的变量dilbert：
```go
type Employee struct {
  ID        int
  Name      string
  Address   string
  DoB       time.Time
  Position  string
  Salary    int
  ManagerID int
}

var dilbert Employee
```
dilbert结构体变量的成员可以通过点操作符访问，比如dilbert.Name和dilbert.DoB。因为dilbert是一个变量，它所有的成员也同样是变量，我们可以直接对每个成员赋值：
```go
dilbert.Salary -= 5000

// 对成员取地址，然后通过指针访问
position := &dilbert.Position
*position = "Senior " + *position

// 点操作符也可以和指向结构体的指针一起工作
var employeeOfTheMonth *Employee = &dilbert
employeeOfTheMonth.Address += " (proactive team player)"

// 等价于
(*employeeOfTheMonth).Address += " (proactive team player)"
```
通常一行对应一个结构体成员，成员的名字在前类型在后，不过如果相邻的成员类型如果相同的话可以被合并到一行，就像下面的Name和Address成员那样：
```go
type Employee struct {
  ID            int
  Name, Address string
  DoB           time.Time
  Position      string
  Salary        int
  ManagerID     int
}
```
结构体成员的输入顺序也有重要的意义。我们也可以将Position成员合并（因为也是字符串类型），或者是交换Name和Address出现的先后顺序，那样的话就是定义了不同的结构体类型。通常，我们只是将相关的成员写到一起。

如果结构体成员名字是以大写字母开头的，那么该成员就是导出的；这是Go语言导出规则决定的。一个结构体可能同时包含导出和未导出的成员

#### 结构体字面值
结构体值也可以用结构体字面值表示，结构体字面值可以指定每个成员的值。
```go
type Point struct{ X, Y int }

p := Point{1, 2}  // 需要按顺序指定值

q := Point{Y: 2, X: 1} // 常用这种写法，以成员名字和相应的值来初始化，可以包含部分或全部的成员
```
因为结构体通常通过指针处理，可以用下面的写法来创建并初始化一个结构体变量，并返回结构体的地址：
```go
pp := &Point{1, 2}

// 等价于
pp := new(Point)
*pp = Point{1, 2}
```
不过&Point{1, 2}写法可以直接在表达式中使用，比如一个函数调用。

#### 结构体比较
如果结构体的全部成员都是可以比较的，那么结构体也是可以比较的，那样的话两个结构体将可以使用==或!=运算符进行比较。相等比较运算符==将比较两个结构体的每个成员，因此下面两个比较的表达式是等价的：
```go
type Point struct{ X, Y int }

p := Point{1, 2}
q := Point{2, 1}
fmt.Println(p.X == q.X && p.Y == q.Y) // "false"
fmt.Println(p == q)                   // "false"
```
#### 结构体嵌入和匿名成员
结构体嵌入允许一个结构体包含另一个结构体的字段，并使用点号访问该结构体的字段。
```go
type Point struct {
  X, Y int
}

type Circle struct {
  Center Point
  Radius int
}

type Wheel struct {
  Circle Circle
  Spokes int
}
var w Wheel
w.Circle.Center.X = 8
w.Circle.Center.Y = 8
w.Circle.Radius = 5
w.Spokes = 20
```
匿名成员允许一个结构体包含一个字段，但不需要使用点号访问该字段。即只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员。匿名成员的数据类型必须是命名的类型或指向一个命名的类型的指针。
```go
type Circle struct {
  Point
  Radius int
}

type Wheel struct {
  Circle
  Spokes int
}

var w Wheel
w.X = 8            // 等价于 w.Circle.Point.X = 8
w.Y = 8            // 等价于 w.Circle.Point.Y = 8
w.Radius = 5       // 等价于 w.Circle.Radius = 5
w.Spokes = 20
```
结构字面值没有简短表示匿名成员的语法，因此以下语句无效：
```go
w = Wheel{8, 8, 5, 20}                       // compile error: unknown fields
w = Wheel{X: 8, Y: 8, Radius: 5, Spokes: 20} // compile error: unknown fields
```
结构体字面值必须遵循形状类型声明时的结构，所以只能用下面的两种语法，它们彼此是等价的：
```go
w = Wheel{Circle{Point{8, 8}, 5}, 20}

w = Wheel{
  Circle: Circle{
    Point:  Point{X: 8, Y: 8},
    Radius: 5,
  },
  Spokes: 20, // NOTE: trailing comma necessary here (and at Radius)
}

fmt.Printf("%#v\n", w)
// Output:
// Wheel{Circle:Circle{Point:Point{X:8, Y:8}, Radius:5}, Spokes:20}

w.X = 42

fmt.Printf("%#v\n", w)
// Output:
// Wheel{Circle:Circle{Point:Point{X:42, Y:8}, Radius:5}, Spokes:20}
```
> Printf函数中%v参数包含的#副词，它表示用和Go语言类似的语法打印值。对于结构体类型来说，将包含每个成员的名字。

### JSON
JavaScript对象表示法（JSON）是一种用于发送和接收结构化信息的标准协议。标准库中提供了encoding/json包，用于解析JSON数据。 
类似的协议有XML(encoding/xml)、ASN.1(encoding/asn1)、YAML(gopkg.in/yaml.v2)、Protocol Buffers(github.com/golang/protobuf)。

基本的JSON类型有数字（十进制或科学记数法）、布尔值（true或false）、字符串，其中字符串是以双引号包含的Unicode字符序列，支持和Go语言类似的反斜杠转义特性，不过JSON使用的是\Uhhhh转义数字来表示一个UTF-16编码（译注：UTF-16和UTF-8一样是一种变长的编码，有些Unicode码点较大的字符需要用4个字节表示；而且UTF-16还有大端和小端的问题），而不是Go语言的rune类型。

Go语言中将结构体slice转为JSON的过程叫编组（marshaling）。编组通过调用json.Marshal函数完成：
```go
var w Wheel
data, err := json.Marshal(w)
if err != nil {
  log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
```
json.MarshalIndent函数将产生整齐缩进的输出。该函数有两个额外的字符串参数用于表示每一行输出的前缀和每一个层级的缩进：
```go
data, err := json.MarshalIndent(w, "", "    ")
if err != nil {
  log.Fatalf("JSON marshaling failed: %s", err)
}
fmt.Printf("%s\n", data)
```

结构体的成员Tag可以是任意的字符串面值，但是通常是一系列用空格分隔的key:"value"键值对序列；因为值中含有双引号字符，因此成员Tag一般用原生字符串面值的形式书写。json开头键名对应的值用于控制encoding/json包的编码和解码的行为，并且encoding/...下面其它的包也遵循这个约定。成员Tag中json对应值的第一部分用于指定JSON对象的名字，比如将Go语言中的TotalCount成员对应到JSON中的total_count对象。Color成员的Tag还带了一个额外的omitempty选项，表示当Go语言结构体成员为空或零值时不生成该JSON对象（这里false为零值）。
```go
Year  int  `json:"released"`
Color bool `json:"color,omitempty"`
```
编码的逆操作是解码，对应将JSON数据解码为Go语言的数据结构，Go语言中一般叫unmarshaling，通过json.Unmarshal函数完成。下面的代码将JSON格式的电影数据解码为一个结构体slice，结构体中只有Title成员。通过定义合适的Go语言数据结构，我们可以选择性地解码JSON中感兴趣的成员。当Unmarshal函数调用返回，slice将被只含有Title信息的值填充，其它JSON成员将被忽略
```go
var titles []struct{ Title string }
if err := json.Unmarshal(data, &titles); err != nil {
  log.Fatalf("JSON unmarshaling failed: %s", err)
}
fmt.Println(titles) // "[{Casablanca} {Cool Hand Luke} {Bullitt}]"
```
基于流式的解码器json.Decoder，它可以从一个输入流解码JSON数据;还有一个针对输出流的json.Encoder编码对象。

### 文本和HTML模板
text/template和html/template等模板包提供了一个将变量值填充到一个文本或HTML格式的模板的机制。

一个模板是一个字符串或一个文件，里面包含了一个或多个由双花括号包含的`{{action}}`对象。大部分的字符串只是按字面值打印，但是对于actions部分将触发其它的行为。
```go
const templ = `{{.TotalCount}} issues:
{{range .Items}}----------------------------------------
Number: {{.Number}}
User:   {{.User.Login}}
Title:  {{.Title | printf "%.64s"}}
Age:    {{.CreatedAt | daysAgo}} days
{{end}}`
```
对于每一个action，都有一个当前值的概念，对应点操作符，写作“.”。当前值“.”最初被初始化为调用模板时的参数，在当前例子中对应github.IssuesSearchResult类型的变量。模板中`{{.TotalCount}}`对应action将展开为结构体中TotalCount成员以默认的方式打印的值。模板中`{{range .Items}}`和`{{end}}`对应一个循环action，因此它们之间的内容可能会被展开多次，循环每次迭代的当前值对应当前的Items元素的值。

在一个action中，|操作符表示将前一个表达式的结果作为后一个函数的输入，类似于UNIX中管道的概念。在Title这一行的action中，第二个操作是一个printf函数，是一个基于fmt.Sprintf实现的内置函数，所有模板都可以直接使用。对于Age部分，第二个动作是一个叫daysAgo的函数，通过time.Since函数将CreatedAt成员转换为过去的时间长度：
```go
func daysAgo(t time.Time) int {
  return int(time.Since(t).Hours() / 24)
}
```
生成模板的输出需要两个处理步骤。第一步是要分析模板并转为内部表示，然后基于指定的输入执行模板。分析模板部分一般只需要执行一次。下面的代码创建并分析上面定义的模板templ。注意方法调用链的顺序：template.New先创建并返回一个模板；Funcs方法将daysAgo等自定义函数注册到模板中，并返回模板；最后调用Parse函数分析模板。
```go
report, err := template.New("report").
  Funcs(template.FuncMap{"daysAgo": daysAgo}).
  Parse(templ)
if err != nil {
  log.Fatal(err)
}
```
因为模板通常在编译时就测试好了，如果模板解析失败将是一个致命的错误。template.Must辅助函数可以简化这个致命错误的处理：它接受一个模板和一个error类型的参数，检测error是否为nil（如果不是nil则发出panic异常），然后返回传入的模板。

```go
var report = template.Must(template.New("issuelist").
  Funcs(template.FuncMap{"daysAgo": daysAgo}).
  Parse(templ))
```

html/template模板包。它使用和text/template包相同的API和模板语言，但是增加了一个将字符串自动转义特性，这可以避免输入字符串和HTML、JavaScript、CSS或URL语法产生冲突的问题。这个特性还可以避免一些长期存在的安全问题，比如通过生成HTML注入攻击，通过构造一个含有恶意代码的问题标题，这些都可能让模板输出错误的输出，从而让他们控制页面。
```go
import "html/template"

var issueList = template.Must(template.New("issuelist").Parse(`
<h1>{{.TotalCount}} issues</h1>
<table>
<tr style='text-align: left'>
  <th>#</th>
  <th>State</th>
  <th>User</th>
  <th>Title</th>
</tr>
{{range .Items}}
<tr>
  <td><a href='{{.HTMLURL}}'>{{.Number}}</a></td>
  <td>{{.State}}</td>
  <td><a href='{{.User.HTMLURL}}'>{{.User.Login}}</a></td>
  <td><a href='{{.HTMLURL}}'>{{.Title}}</a></td>
</tr>
{{end}}
</table>
`))
```
可以通过对信任的HTML字符串使用template.HTML类型来抑制这种自动转义的行为。
```go
func main() {
  const templ = `<p>A: {{.A}}</p><p>B: {{.B}}</p>`
  t := template.Must(template.New("escape").Parse(templ))
  var data struct {
    A string        // untrusted plain text
    B template.HTML // trusted HTML
  }
  data.A = "<b>Hello!</b>"
  data.B = "<b>Hello!</b>"
  if err := t.Execute(os.Stdout, data); err != nil {
    log.Fatal(err)
  }
}
```

## 函数
函数可以让我们将一个语句序列打包为一个单元，然后可以从程序中其它地方多次调用。函数的机制可以让我们将一个大的工作分解为小的任务，这样的小任务可以让不同程序员在不同时间、不同地方独立完成。一个函数同时对用户隐藏了其实现细节。由于这些因素，对于任何编程语言来说，函数都是一个至关重要的部分

### 函数声明
函数声明包括函数名、形式参数列表、返回值列表（可省略）以及函数体。
```go
func name(parameter-list) (result-list) {
  body
}
```
形参列表描述了函数的参数名和参数类型，属于局部变量。返回值列表描述了函数返回值的变量名以及类型，如果返回一个无名变量或者没有返回值，返回值列表可以省略。
如果一组形参或返回值有相同的类型，可以不用写出每一个参数类型
```go
func add(x int, y int) int   {return x + y}
func sub(x, y int) (z int)   { z = x - y; return}
func first(x int, _ int) int { return x }
func zero(int, int) int      { return 0 }

fmt.Printf("%T\n", add)   // "func(int, int) int"
fmt.Printf("%T\n", sub)   // "func(int, int) int"
fmt.Printf("%T\n", first) // "func(int, int) int"
fmt.Printf("%T\n", zero)  // "func(int, int) int"
```
实参通过值的方式传递，因此函数的形参是实参的拷贝。对形参进行修改不会影响实参。但是，如果实参包括引用类型，如指针，slice(切片)、map、function、channel等类型，实参可能会由于函数的间接引用被修改。

### 错误
在Go的错误处理中，错误是软件包API和应用程序用户界面的一个重要组成部分，程序运行失败仅被认为是几个预期的结果之一，对于那些将运行失败看作是预期结果的函数，它们会返回一个额外的返回值，通常是最后一个，来传递错误信息。如果导致失败的原因只有一个，额外的返回值可以是一个布尔值，通常被命名为ok。

在Go中，函数运行失败时会返回错误信息，这些错误信息被认为是一种预期的值而非异常（exception），这使得Go有别于那些将函数运行失败看作是异常的语言。虽然Go有各种异常机制，但这些机制仅被使用在处理那些未被预料到的错误，即bug，而不是那些在健壮程序中应该被避免的程序错误。

#### 错误处理策略
当一次函数调用返回错误时，调用者应该选择合适的方式处理错误。根据情况的不同，有很多处理方式，以下是常用的五种方式。

1. 传播错误；这意味着函数中某个子程序的失败，会变成该函数的失败。
```go
resp, err := http.Get(url)
if err != nil{
  return nil, err  // 直接返回错误
}
```
当对html.Parse的调用失败时，findLinks不会直接返回html.Parse的错误，因为缺少两条重要信息：1、发生错误时的解析器（html parser）；2、发生错误的url。因此，findLinks构造了一个新的错误信息，既包含了这两项，也包括了底层的解析出错的信息。
```go
doc, err := html.Parse(resp.Body)
resp.Body.Close()
if err != nil {
  return nil, fmt.Errorf("parsing %s as HTML: %v", url,err)  // 构造错误信息然后返回
}
```
编写错误信息时，我们要确保错误信息对问题细节的描述是详尽的。尤其是要注意错误信息表达的一致性，即相同的函数或同包内的同一组函数返回的错误在构成和处理方式上是相似的。

2. 重试操作；如果错误的发生是偶然性的，或由不可预知的问题导致的。一个明智的选择是重新尝试失败的操作。在重试时，我们需要限制重试的时间间隔或重试的次数，防止无限制的重试。
```go
// WaitForServer 尝试联系 URL 的服务器。它使用指数退避算法尝试一分钟。如果所有尝试都失败，则会报告错误。
func WaitForServer(url string) error {
  const timeout = 1 * time.Minute
  deadline := time.Now().Add(timeout)
  for tries := 0; time.Now().Before(deadline); tries++ {
    _, err := http.Head(url)
    if err == nil {
      return nil // success
    }
    log.Printf("server not responding (%s);retrying…", err)
    time.Sleep(time.Second << uint(tries)) // exponential back-off
  }
  return fmt.Errorf("server %s failed to respond after %s", url, timeout)
}
```

3. 输出错误信息并结束程序;如果错误发生后，程序无法继续运行时采用，这种策略只应在main中执行。对库函数而言，应仅向上传播错误，除非该错误意味着程序内部包含不一致性，即遇到了bug，才能在库函数中结束程序。
```go
// 在main函数里
if err := WaitForServer(url); err != nil {
  fmt.Fprintf(os.Stderr, "Site is down: %v\n", err)
  
  // 或者
  log.Fatalf("Site is down: %v\n", err) // log中的所有函数，都默认会在错误信息之前输出时间信息。

  os.Exit(1)
}
```
可以设置log的前缀信息屏蔽时间信息，一般而言，前缀信息会被设置成命令名。
```go
log.SetPrefix("wait: ")
log.SetFlags(0)
```

4. 输出错误信息；输出错误信息，不需要中断程序的运行。
```go
if err := Ping(); err != nil {
  // log包中的所有函数会为没有换行符的字符串增加换行符。
  log.Printf("ping failed: %v; networking disabled",err)
}

// 或者标准错误流输出错误信息。
if err := Ping(); err != nil {
  fmt.Fprintf(os.Stderr, "ping failed: %v; networking disabled\n", err)
}
```

5. 忽略错误；
```go
dir, err := ioutil.TempDir("", "scratch")
if err != nil {
    return fmt.Errorf("failed to create temp dir: %v",err)
}
// ...use temp dir…
os.RemoveAll(dir) // ignore errors; $TMPDIR is cleaned periodically
```
尽管os.RemoveAll会失败，但上面的例子并没有做错误处理。这是因为操作系统会定期的清理临时目录。正因如此，虽然程序没有处理错误，但程序的逻辑不会因此受到影响。我们应该在每次函数调用后，都养成考虑错误处理的习惯，当你决定忽略某个错误时，你应该清晰地写下你的意图。

### 函数值
在Go中，函数被看作第一类值（first-class values）：函数像其他值一样，拥有类型，可以被赋值给其他变量，传递给函数，从函数返回。对函数值（function value）的调用类似函数调用。
```go
func square(n int) int { return n * n }
  func negative(n int) int { return -n }
  func product(m, n int) int { return m * n }

  f := square
  fmt.Println(f(3)) // "9"

  f = negative
  fmt.Println(f(3))     // "-3"
  fmt.Printf("%T\n", f) // "func(int) int"

  f = product // compile error: can't assign func(int, int) int to func(int) int
```
函数类型的零值是nil。调用值为nil的函数值会引起panic错误：
```go
  var f func(int) int
  f(3) // 此处f的值为nil, 会引起panic错误
```
函数值可以与nil比较：
```go
  var f func(int) int
  if f != nil {
    f(3)
  }
```
但是函数值之间是不可比较的，也不能用函数值作为map的key。

函数值使得我们不仅仅可以通过数据来参数化函数，亦可通过行为。标准库中包含许多这样的例子。下面的代码展示了如何使用这个技巧。strings.Map对字符串中的每个字符调用add1函数，并将每个add1函数的返回值组成一个新的字符串返回给调用者。
```go
  func add1(r rune) rune { return r + 1 }

  fmt.Println(strings.Map(add1, "HAL-9000")) // "IBM.:111"
  fmt.Println(strings.Map(add1, "VMS"))      // "WNT"
  fmt.Println(strings.Map(add1, "Admix"))    // "Benjy"
```

### 匿名函数
拥有函数名的函数只能在包级语法块中被声明，通过函数字面量（function literal），我们可绕过这一限制，在任何表达式中表示一个函数值。函数字面量的语法和函数声明相似，区别在于func关键字后没有函数名。函数值字面量是一种表达式，它的值被称为匿名函数（anonymous function）。

```go
// 函数字面量允许我们在使用函数时，再定义它。
strings.Map(func(r rune) rune { return r + 1 }, "HAL-9000")

// squares返回一个匿名函数。该匿名函数每次被调用时都会返回下一个数的平方。
func squares() func() int {
  var x int
  return func() int {
    x++
    return x * x
  }
}
func main() {
  f := squares()
  fmt.Println(f()) // "1"
  fmt.Println(f()) // "4"
  fmt.Println(f()) // "9"
  fmt.Println(f()) // "16"
}
```
当匿名函数需要被递归调用时，必须首先声明一个变量（在下面的例子中，我们首先声明了 visitAll），再将匿名函数赋值给这个变量。如果不分成两步，函数字面量无法与visitAll绑定，也就无法递归调用该匿名函数。
```go
var visitAll func(items []string)
visitAll = func(items []string) {/** */}

// 这样会报错
visitAll := func(items []string) {
  // ...
  visitAll(m[item]) // compile error: undefined: visitAll
  // ...
}
```

### 可变参数
参数数量可变的函数称为可变参数函数。典型的例子就是fmt.Printf和类似函数。Printf首先接收一个必备的参数，之后接收任意个数的后续参数。

在声明可变参数函数时，需要在参数列表的最后一个参数类型之前加上省略符号“...”，这表示该函数会接收任意数量的该类型参数。
```go
// 返回任意个int型参数的和
func sum(vals ...int) int {
  total := 0
  for _, val := range vals {
    total += val
  }
  return total
}

fmt.Println(sum())           // "0"
fmt.Println(sum(3))          // "3"
fmt.Println(sum(1, 2, 3, 4)) // "10"

// 传递切片
values := []int{1, 2, 3, 4}
fmt.Println(sum(values...)) // "10"
```

虽然在可变参数函数内部，...int 型参数的行为看起来很像切片类型，但实际上，可变参数函数和以切片作为参数的函数是不同的。
```go
func f(...int) {}
func g([]int) {}
fmt.Printf("%T\n", f) // "func(...int)"
fmt.Printf("%T\n", g) // "func([]int)"
```

### Deferred函数
Deferred函数是延迟执行，当函数返回时，会先执行Deferred函数。

在调用普通函数或方法前加上关键字defer，就完成了defer所需要的语法。当执行到该条语句时，函数和参数表达式得到计算，但直到包含该defer语句的函数执行完毕时，defer后的函数才会被执行，不论包含defer语句的函数是通过return正常结束，还是由于panic导致的异常结束。你可以在一个函数中执行多条defer语句，它们的执行顺序与声明顺序相反。

defer语句经常被用于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。释放资源的defer应该直接跟在请求资源的语句后。

被延迟执行的匿名函数甚至可以修改函数返回给调用者的返回值：
```go
func triple(x int) (result int) {
  defer func() { result += x }()
  return x + x
}
fmt.Println(triple(4)) // "12"
```

### Panic异常
Go的类型系统会在编译时捕获很多错误，但有些错误只能在运行时检查，如数组访问越界、空指针引用等。这些运行时错误会引起painc异常。 
panic是来自被调用函数的信号，表示发生了某个已知的bug。一个良好的程序永远不应该发生panic异常。

一般而言，当panic异常发生时，程序会中断运行，并立即执行在该goroutine中被延迟的函数（defer 机制）。随后，程序崩溃并输出日志信息。日志信息包括panic value和函数调用的堆栈跟踪信息。panic value通常是某种错误信息。对于每个goroutine，日志信息中都会有与之相对的，发生panic时的函数调用堆栈跟踪信息。

不是所有的panic异常都来自运行时，直接调用内置的panic函数也会引发panic异常；panic函数接受任何值作为参数。当某些不应该发生的场景发生时，我们就应该调用panic。比如，当程序到达了某条逻辑上不可能到达的路径
```go
switch s := suit(drawCard()); s {
case "Spades":                                // ...
case "Hearts":                                // ...
case "Diamonds":                              // ...
case "Clubs":                                 // ...
default:
  panic(fmt.Sprintf("invalid suit %q", s)) // Joker?
}
```

虽然Go的panic机制类似于其他语言的异常，但panic的适用场景有一些不同。由于panic会引起程序的崩溃，因此panic一般用于严重错误，如程序内部的逻辑不一致。对于大部分漏洞，我们应该使用Go提供的错误机制，而不是panic，尽量避免程序的崩溃。

```go
func main() {
  defer printStack()
  f(3)
}
func f(x int) {
  fmt.Printf("f(%d)\n", x+0/x) // panics if x == 0
  defer fmt.Printf("defer %d\n", x)
  f(x - 1)
}
func printStack() {
  var buf [4096]byte
  n := runtime.Stack(buf[:], false) // 获取调用栈
  os.Stdout.Write(buf[:n])
}
```

### Recover捕获异常
通常来说，不应该对panic异常做任何处理，但有时，也许我们可以从异常中恢复，至少我们可以在程序崩溃前，做一些操作。举个例子，当web服务器遇到不可预料的严重问题时，在崩溃前应该将所有的连接关闭；如果不做任何处理，会使得客户端一直处于等待状态。如果web服务器还在开发阶段，服务器甚至可以将异常信息反馈到客户端，帮助调试。

如果在deferred函数中调用了内置函数recover，并且定义该defer语句的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil。

```go
func Parse(input string) (s *Syntax, err error) {
    defer func() {
      if p := recover(); p != nil {
        err = fmt.Errorf("internal error: %v", p)
      }
  }()
  // ...parser...
}
```
deferred函数帮助Parse从panic中恢复。在deferred函数内部，panic value被附加到错误信息中；并用err变量接收错误信息，返回给调用者。我们也可以通过调用runtime.Stack往错误信息中添加完整的堆栈调用信息。

不加区分的恢复所有的panic异常，不是可取的做法；因为在panic之后，无法保证包级变量的状态仍然和我们预期一致。比如，对数据结构的一次重要更新没有被完整完成、文件或者网络连接没有被关闭、获得的锁没有被释放。此外，如果写日志时产生的panic被不加区分的恢复，可能会导致漏洞被忽略。

虽然把对panic的处理都集中在一个包下，有助于简化对复杂和不可以预料问题的处理，但作为被广泛遵守的规范，你不应该试图去恢复其他包引起的panic。公有的API应该将函数的运行失败作为error返回，而不是panic。同样的，你也不应该恢复一个由他人开发的函数引起的panic，比如说调用者传入的回调函数，因为你无法确保这样做是安全的。

有时我们很难完全遵循规范，举个例子，net/http包中提供了一个web服务器，将收到的请求分发给用户提供的处理函数。很显然，我们不能因为某个处理函数引发的panic异常，杀掉整个进程；web服务器遇到处理函数导致的panic时会调用recover，输出堆栈信息，继续运行。这样的做法在实践中很便捷，但也会引起资源泄漏，或是因为recover操作，导致其他问题。

基于以上原因，安全的做法是有选择性的recover。换句话说，只恢复应该被恢复的panic异常，此外，这些异常所占的比例应该尽可能的低。为了标识某个panic是否应该被恢复，我们可以将panic value设置成特殊类型。在recover时对panic value进行检查，如果发现panic value是特殊类型，就将这个panic作为error处理，如果不是，则按照正常的panic进行处理。

## 方法

### 方法声明
在函数声明时，在其名字之前放上一个变量，即是一个方法。这个附加的参数会将该函数附加到这种类型上，即相当于为这种类型定义了一个独占的方法。
``` go
package geometry

import "math"

type Point struct{ X, Y float64 }

// 传统方法
func Distance(p, q Point) float64 {
  return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// 同样的事情，但作为 Point 类型的方法
func (p Point) Distance(q Point) float64 {
  return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```
上面的代码里那个附加的参数p，叫做方法的接收器（receiver），在Go语言中，没有像其它语言那样用this或者self作为接收器；我们可以任意的选择接收器的名字。由于接收器的名字经常会被使用到，所以保持其在方法间传递时的一致性和简短性是不错的主意。这里的建议是可以使用其类型的第一个字母，比如这里使用了Point的首字母p。

在方法调用过程中，接收器参数一般会在方法名之前出现。这和方法声明是一样的，都是接收器参数在方法名字之前。
```go
p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // "5", 用的是包级别的函数geometry.Distance
fmt.Println(p.Distance(q))  // "5", 调用的是Point类下声明的Point.Distance方法。
```
> 有别于其他语言，Go语言中，可以给任意类型定义方法

### 基于指针对象的方法
当调用一个函数时，会对其每一个参数值进行拷贝，如果一个函数需要更新一个变量，或者函数的其中一个参数实在太大我们希望能够避免进行这种默认的拷贝，这种情况下就需要用到指针了。对应到我们这里用来更新接收器的对象的方法，当这个接受者变量本身比较大时，我们就可以用其指针而不是对象来声明方法，如下：
```go
func (p *Point) ScaleBy(factor float64) {
  p.X *= factor
  p.Y *= factor
}
```
这个方法的名字是`(*Point).ScaleBy`。这里的括号是必须的；没有括号的话这个表达式可能会被理解为`*(Point.ScaleBy)`。

在现实的程序里，一般会约定如果Point这个类有一个指针作为接收器的方法，那么所有Point的方法都必须有一个指针接收器，即使是那些并不需要这个指针接收器的函数。

只有类型（Point）和指向他们的指针(*Point)，才可能是出现在接收器声明里的两种接收器。此外，为了避免歧义，在声明方法时，如果一个类型名本身是一个指针的话，是不允许其出现在接收器中的。
```go
type P *int
func (P) f() { /* ... */ } // compile error: invalid receiver type
```
想要调用指针类型方法(*Point).ScaleBy，只要提供一个Point类型的指针即可。
```go
r := &Point{1, 2}
r.ScaleBy(2)
fmt.Println(*r) // "{2, 4}"

// 或者
p := Point{1, 2}
pptr := &p
pptr.ScaleBy(2)
fmt.Println(p) // "{2, 4}"

// 或者
p := Point{1, 2}
(&p).ScaleBy(2)
fmt.Println(p) // "{2, 4}"
```
如果接收器p是一个Point类型的变量，并且其方法需要一个Point指针作为接收器，可以用下面这种简短的写法：
```go
p.ScaleBy(2)
```
编译器会隐式地帮我们用&p去调用ScaleBy这个方法。这种简写方法只适用于“变量”，包括struct里的字段比如p.X，以及array和slice内的元素比如perim[0]。不能通过一个无法取到地址的接收器来调用指针方法，比如临时变量的内存地址就无法获取得到：
```go
Point{1, 2}.ScaleBy(2) // compile error: can't take address of Point literal
```
在每一个合法的方法调用表达式中，满足下面三种情况里的任意一种情况都是可以的：
1. 接收器的实际参数和其形式参数是相同的类型，比如两者都是类型T或者都是类型`*T`:
```go
Point{1, 2}.Distance(q) //  Point
pptr.ScaleBy(2)         // *Point
```
2. 接收器实参是类型T，但接收器形参是类型 `*T`，这种情况下编译器会隐式地取变量的地址：
```go
p.ScaleBy(2) // implicit (&p)
```
3. 接收器实参是类型`*T`，形参是类型T。编译器会隐式地解引用，取到指针指向的实际变量：
```go
pptr.Distance(q) // implicit (*pptr)
```
**总结**：
1. 不管method的receiver是指针类型还是非指针类型，都是可以通过指针/非指针类型进行调用的，编译器会做类型转换。
2. 在声明一个method的receiver该是指针还是非指针类型时，需要考虑两方面的因素，第一方面是这个对象本身是不是特别大，如果声明为非指针变量时，调用会产生一次拷贝；第二方面是如果用指针类型作为receiver，那么一定要注意，这种指针类型指向的始终是一块内存地址，就算对其进行了拷贝。

#### Nil也是一个合法的接收器类型
就像一些函数允许nil指针作为参数一样，方法理论上也可以用nil指针作为其接收器，尤其当nil对于对象来说是合法的零值时，比如map或者slice。在下面的简单int链表的例子里，nil代表的是空链表：
```go
// IntList 是一个整数链表。 nil *IntList 表示空列表。
type IntList struct {
  Value int
  Tail  *IntList
}
// Sum 返回列表元素的总和。
func (list *IntList) Sum() int {
  if list == nil {
    return 0
  }
  return list.Value + list.Tail.Sum()
}
```
当定义一个允许nil作为接收器值的方法的类型时，在类型前面的注释中指出nil变量代表的意义是很有必要的。

### 通过嵌入结构体来扩展类型
在一个结构体中 直接嵌入另一个结构体（或接口），被嵌入的结构体的字段和方法会自动提升（Promoted）到外层结构体。 
语法：直接写嵌入结构体的名称（无需字段名）。
```go
type Person struct {
  Name string
  Age  int
}

// Employee 嵌入 Person
type Employee struct {
  Person      // 直接嵌入（无字段名）
  Company string
}
```
此时：`Employee` **自动拥有** `Person` 的**所有字段**（`Name` 和 `Age`）。可以通过 `Employee` 直接访问 `Person` 的字段和方法，就像它们是 `Employee` 的一部分。

#### 嵌入后的行为
**字段和方法的提升（Promotion）**: 嵌入的结构体的字段和方法会被 **“提升”** 到外层结构体:
```go
emp := Employee{
    Person: Person{Name: "Alice", Age: 30},
    Company: "Google",
}

fmt.Println(emp.Name) // 直接访问 Person 的字段（无需 emp.Person.Name）
```
等价于 `emp.Person.Name`，但语法更简洁。

**方法继承**: 如果被嵌入的结构体有方法，外层结构体也会继承这些方法：
```go
func (p Person) Greet() {
    fmt.Printf("Hello, I'm %s\n", p.Name)
}

emp := Employee{Person: Person{Name: "Bob"}}
emp.Greet() // 输出: Hello, I'm Bob
```
#### 嵌入的冲突处理
**字段或方法重名** 
如果外层结构体和嵌入结构体有同名字段或方法：
- 外层结构体的字段/方法 优先。
- 需要显式指定嵌入结构体名来访问被覆盖的字段/方法：
```go
type A struct { X int }
type B struct { X int; A }

b := B{X: 1, A: A{X: 2}}
fmt.Println(b.X)      // 1（外层优先）
fmt.Println(b.A.X)    // 2（显式指定）
```
**多重嵌入的歧义** 
如果多个嵌入结构体有同名字段/方法，必须显式指定：
```go
type A struct { X int }
type B struct { X int }
type C struct { A; B }

c := C{A: A{X: 1}, B: B{X: 2}}
fmt.Println(c.A.X) // 必须明确指定 A 或 B
```

#### 嵌入接口
**嵌入接口的类型** 
一个结构体可以嵌入 接口类型，此时该结构体必须实现接口的所有方法（否则编译错误）：
```go
type Writer interface {
  Write([]byte) (int, error)
}

type LogWriter struct {
  Writer // 嵌入接口
}

// LogWriter 必须实现 Writer 接口
func (lw LogWriter) Write(data []byte) (int, error) {
  return fmt.Println("Log:", string(data))
}
```
**用途** 
- 装饰器模式：通过嵌入接口扩展原有实现。
- 依赖注入：动态替换嵌入接口的具体实现。

### 封装
一个对象的变量或者方法如果对调用方是不可见的话，一般就被定义为“封装”。封装有时候也被叫做信息隐藏，同时也是面向对象编程最关键的一个方面。

Go语言只有一种控制可见性的手段：大写首字母的标识符会从定义它们的包中被导出，小写字母的则不会。这种限制包内成员的方式同样适用于struct或者一个类型的方法。因而如果我们想要封装一个对象，我们必须将其定义为一个struct。
```go
type IntSet struct {
  words []uint64
}
```

封装的优点：
1. 因为调用方不能直接修改对象的变量值，其只需要关注少量的语句并且只要弄懂少量变量的可能的值即可。
2. 隐藏实现的细节，可以防止调用方依赖那些可能变化的具体实现，这样使设计包的程序员在不破坏对外的api情况下能得到更大的自由。
3. 阻止了外部调用方对对象内部的值任意地进行修改。

## 接口
接口类型是对其它类型行为的抽象和概括；因为接口类型不会和特定的实现细节绑定在一起，通过这种抽象的方式可以让函数更加灵活和更具有适应能力。

Go语言中接口类型的独特之处在于它是满足隐式实现的。也就是说，我们没有必要对于给定的具体类型定义所有满足的接口类型；简单地拥有一些必需的方法就足够了。这种设计可以让你创建一个新的接口类型满足已经存在的具体类型却不会去改变这些类型的定义；当使用的类型来自于不受控制的包时这种设计尤其有用。

### 接口约定
接口类型是一种抽象的类型。它不会暴露出它所代表的对象的内部值的结构和这个对象支持的基础操作的集合；它们只会表现出它们自己的方法。也就是说当你有看到一个接口类型的值时，你不知道它是什么，唯一知道的就是可以通过它的方法来做什么。

fmt.Printf 和 fmt.Sprintf 这两个函数使用了 fmt.Fprintf 来进行封装。
```go
package fmt

func Fprintf(w io.Writer, format string, args ...interface{}) (int, error)
func Printf(format string, args ...interface{}) (int, error) {
  return Fprintf(os.Stdout, format, args...)
}
func Sprintf(format string, args ...interface{}) string {
  var buf bytes.Buffer
  Fprintf(&buf, format, args...)
  return buf.String()
}
```
Fprintf函数中的第一个参数也不是一个文件类型。它是io.Writer类型，这是一个接口类型定义如下：
```go
package io

// Writer 是包装基本 Write 方法的接口
type Writer interface {
  // Write 将 len(p) 个字节从 p 写入底层数据流。它返回从 p 写入的字节数 (0 <= n <= len(p))以及导致写入提前停止的任何错误。
  // 如果返回 n < len(p)，Write 必须返回非零错误。Write 不得修改切片数据，即使是暂时的。
  //
  // 实现不得保留 p。
  Write(p []byte) (n int, err error)
}
```
io.Writer类型定义了函数Fprintf和这个函数调用者之间的约定。一方面这个约定需要调用者提供具体类型的值就像*os.File和*bytes.Buffer，这些类型都有一个特定签名和行为的Write的函数。另一方面这个约定保证了Fprintf接受任何满足io.Writer接口的值都可以工作。Fprintf函数可能没有假定写入的是一个文件或是一段内存，而是写入一个可以调用Write函数的值。

因为fmt.Fprintf函数没有对具体操作的值做任何假设，而是仅仅通过io.Writer接口的约定来保证行为，所以第一个参数可以安全地传入一个只需要满足io.Writer接口的任意具体类型的值。一个类型可以自由地被另一个满足相同接口的类型替换，被称作可替换性（LSP里氏替换）。这是一个面向对象的特征。

### 接口类型
接口类型具体描述了一系列方法的集合，一个实现了这些方法的具体类型是这个接口类型的实例。

io.Writer类型是用得最广泛的接口之一，因为它提供了所有类型的写入bytes的抽象，包括文件类型，内存缓冲区，网络链接，HTTP客户端，压缩工具，哈希等等。io包中定义了很多其它有用的接口类型。Reader可以代表任意可以读取bytes的类型，Closer可以是任意可以关闭的值，例如一个文件或是网络链接。
```go
package io
type Reader interface {
  Read(p []byte) (n int, err error)
}
type Closer interface {
  Close() error
}
```
通过组合已有接口来创建新的接口，称为**接口内嵌**
```go
type ReadWriter interface {
  Reader
  Writer
}
type ReadWriteCloser interface {
  Reader
  Writer
  Closer
}

// 或者不使用内嵌方式
type ReadWriter interface {
  Read(p []byte) (n int, err error)
  Write(p []byte) (n int, err error)
}

// 或者混合使用
type ReadWriter interface {
  Read(p []byte) (n int, err error)
  Writer
}
```
以上定义的方式效果一样，方法顺序没有影响，唯一重要的时里面的方法。

### 实现接口的条件
一个类型如果拥有一个接口需要的所有方法，那么这个类型就实现了这个接口。例如，`*os.File`类型实现了io.Reader，Writer，Closer，和ReadWriter接口。Go的程序员经常会简要的把一个具体的类型描述成一个特定的接口类型。举个例子，*bytes.Buffer是io.Writer；*os.Files是io.ReadWriter。

接口指定的规则非常简单：表达一个类型属于某个接口只要这个类型实现这个接口。所以：
```go
var w io.Writer
w = os.Stdout           // OK: *os.File 有 Write 方法
w = new(bytes.Buffer)   // OK: *bytes.Buffer 有 Write 方法
w = time.Second         // compile error: time.Duration lacks Write method

var rwc io.ReadWriteCloser
rwc = os.Stdout         // OK: *os.File 有 Read, Write, Close 方法
rwc = new(bytes.Buffer) // compile error: *bytes.Buffer lacks Close method
```
这个规则甚至适用于等式右边本身也是一个接口类型
```go
w = rwc                 // OK: io.ReadWriteCloser 有 Write 方法
rwc = w                 // compile error: io.Writer lacks Close method
```
interface{}被称为空接口类型是不可或缺的。因为空接口类型对实现它的类型没有要求，所以可以将任意一个值赋给空接口类型。

### flag.Value接口
flag.Value 接口是 flag 包提供的一个核心接口，用于 自定义命令行标志（flag）的解析逻辑。通过实现该接口，可以定义非标准类型（如自定义结构体、复杂格式的输入）如何从命令行字符串解析为具体的值。
```go
type Value interface {
  String() string    // 返回值的字符串表示（用于默认值显示）
  Set(string) error  // 从字符串解析值，失败时返回 error
}
```
任何实现了这两个方法的类型，都可以作为 flag.Var() 的参数，用于自定义标志解析。

#### 实现 `flag.Value` 的步骤
**(1) 定义自定义类型**
选择需要解析的目标类型（如 `time.Duration`、`net.IP` 或自定义结构体）。

**(2) 实现 `String()` 和 `Set()` 方法**
- `String()`：返回当前值的字符串形式（用于帮助信息）。
- `Set(string) error`：将输入的字符串解析为内部值。

**(3) 注册到 flag**
使用 `flag.Var()` 将自定义类型绑定到命令行标志。

#### 标准库中的实例
Go 标准库中有多个类型实现了 flag.Value：
| 类型            | 用途         | 示例输入          |
| --------------- | ------------ | ----------------- |
| `time.Duration` | 解析时间间隔 | `-timeout 30s`    |
| `net.IP`        | 解析 IP 地址 | `-ip 192.168.1.1` |
| `fs.FileMode`   | 解析文件权限 | `-mode 0644`      |

#### 对比 `flag.Value` 和 `flag.Flag`
| 特性     | `flag.Value`     | `flag.Flag`               |
| -------- | ---------------- | ------------------------- |
| 作用     | 定义值的解析逻辑 | 包装一个具体的命令行标志  |
| 实现方式 | 用户实现接口     | 由 `flag` 包内部管理      |
| 典型用途 | 自定义类型解析   | 标准类型（int, string等） |

```go
// 解析坐标点 -point 1,2
type Point struct{ X, Y int }

// 实现 flag.Value 接口
func (p *Point) String() string {
  return fmt.Sprintf("(%d,%d)", p.X, p.Y)
}

func (p *Point) Set(s string) error {
  _, err := fmt.Sscanf(s, "%d,%d", &p.X, &p.Y)
  return err
}

func main() {
  var pt Point
  // \ 注册并使用
  flag.Var(&pt, "point", "coordinate point (e.g., 1,2)")
  flag.Parse()
  fmt.Println("Point:", pt)
}
```

### 接口值
**接口值（Interface Value）** 是接口类型变量的具体表现形式，它由两部分组成：**动态类型（Dynamic Type）** 和 **动态值（Dynamic Value）**。

#### 接口值的内部结构
接口值在内存中分为两部分：
- **动态类型**：存储实现接口的具体类型信息（`_type` 结构）。
- **动态值**：指向实际数据的指针（或直接存储值，若值可被指针化）。

内存布局示意图
```go
var w io.Writer     // 接口变量
w = os.Stdout       // 赋值后
```
```text
+---------------------------+
|          接口值 w          |
+-----------+---------------+
| 动态类型   |     动态值     |
| (*os.File)| 指向 os.Stdout 的指针 |
+-----------+---------------+
```

#### 接口值的两种状态
**非空接口值（Non-nil Interface Value）**
- 动态类型和动态值均不为 `nil`。
-可安全调用接口方法。
```go
var w io.Writer = os.Stdout // 非空接口值
w.Write([]byte("hello"))    // 调用成功
```
**(2) 空接口值（Nil Interface Value）**
- 动态类型和动态值均为 nil。
- 调用方法会触发 panic。
```go
var w io.Writer // 空接口值（nil）
w.Write([]byte("hello")) // panic: nil pointer dereference
```
**(3) 动态值为 nil 的接口值**
- 动态类型非 nil，但动态值为 nil。
- 仍可调用方法（需方法内部处理 nil 逻辑）。
```go
var buf *bytes.Buffer      // buf 是 nil
var w io.Writer = buf      // 动态类型为 *bytes.Buffer，动态值为 nil
w.Write([]byte("hello"))   // 调用成功（但 bytes.Buffer 的 Write 方法需处理 nil）
```

#### 接口值的比较
- **动态类型和动态值均相等时**，接口值才相等。
- `nil` **接口值** 只与 `nil` 相等。
```go
var w1 io.Writer = os.Stdout
var w2 io.Writer = os.Stdout
fmt.Println(w1 == w2) // true（相同动态类型和值）

var w3 io.Writer
fmt.Println(w3 == nil) // true
```
如果两个接口值的动态类型相同，但是这个动态类型是不可比较的（比如切片），将它们进行比较就会失败并且panic:
```go
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // panic: comparing uncomparable type []int
```
接口类型是非常与众不同的。其它类型要么是安全的可比较类型（如基本类型和指针）要么是完全不可比较的类型（如切片，映射类型，和函数），但是在比较接口值或者包含了接口值的聚合类型时，我们必须要意识到潜在的panic。同样的风险也存在于使用接口作为map的键或者switch的操作数。只能比较你非常确定它们的动态值是可比较类型的接口值。

| 特性           | 说明                                                  |
| -------------- | ----------------------------------------------------- |
| 动态类型       | 实现接口的具体类型信息（运行时确定）。                |
| 动态值         | 指向实际数据的指针（或内联存储）。                    |
| 空接口 vs 非空 | 空接口 interface{} 可接受任何值，非空接口需方法匹配。 |
| 类型断言       | 运行时检查接口值的动态类型。                          |
| 比较规则       | 动态类型和值均相等时接口值才相等。                    |

> 一个不包含任何值的nil接口值和一个刚好包含nil指针的接口值是不同的。
> 避免频繁接口转换：接口方法调用比直接调用稍慢（需查表）。

### 标准库接口
这里简单介绍几个内置的标准库接口的用法。

#### sort.Interface接口
sort.Interface接口定义了排序算法需要的方法：一个内置的排序算法需要知道三个东西：序列的长度，表示两个元素比较的结果，一种交换两个元素的方式；
```go
package sort

type Interface interface {
  Len() int
  Less(i, j int) bool // i, j 是序列元素的索引
  Swap(i, j int)
}
```

为了对序列进行排序，需要定义一个实现了这三个方法的类型，然后对这个类型的一个实例应用sort.Sort函数。实现了sort.Interface的具体类型不一定是切片类型，也可以是一个结构体类型。
```go
type StringSlice []string
func (p StringSlice) Len() int           { return len(p) }
func (p StringSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p StringSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

// 调用
sort.Sort(StringSlice(names))

// 简化版本 sort包为[]int、[]string和[]float64的正常排序提供了特定版本的函数和类型
sort.Strings(names)

// 逆序
sort.Sort(sort.Reverse(StringSlice(names)))
```
sort包定义了一个不公开的struct类型reverse，它嵌入了一个sort.Interface。reverse的Less方法调用了内嵌的sort.Interface值的Less方法，但是通过交换索引的方式使排序结果变成逆序。
```go
package sort

type reverse struct{ Interface } // that is, sort.Interface

func (r reverse) Less(i, j int) bool { return r.Interface.Less(j, i) }

func Reverse(data Interface) Interface { return reverse{data} }
```
reverse的另外两个方法Len和Swap隐式地由原有内嵌的sort.Interface提供。因为reverse是一个不公开的类型，所以导出函数Reverse返回一个包含原有sort.Interface值的reverse类型实例。

#### http.Handler接口
http.Handler接口定义了一个 ServeHTTP 方法，该方法接收一个 http.ResponseWriter 和一个 http.Request 类型的参数。
```go
package http

type Handler interface {
  ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```
ListenAndServe函数需要一个例如“localhost:8000”的服务器地址，和一个所有请求都可以分派的Handler接口实例。它会一直运行，直到这个服务因为一个错误而失败（或者启动失败），它的返回值一定是一个非空的错误。

示例：
```go
func main() {
  db := database{"shoes": 50, "socks": 5}
  log.Fatal(http.ListenAndServe("localhost:8000", db))
}

type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars

func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
  for item, price := range db {
    fmt.Fprintf(w, "%s: %s\n", item, price)
  }
}
```
net/http包提供了一个请求多路器ServeMux来简化URL和handlers的联系。一个ServeMux将一批http.Handler聚集到一个单一的http.Handler中。
```go
func main() {
  db := database{"shoes": 50, "socks": 5}
  mux := http.NewServeMux()
  mux.Handle("/list", http.HandlerFunc(db.list))
  mux.Handle("/price", http.HandlerFunc(db.price))
  log.Fatal(http.ListenAndServe("localhost:8000", mux))
}

type database map[string]dollars

func (db database) list(w http.ResponseWriter, req *http.Request) {
	for item, price := range db {
		fmt.Fprintf(w, "%s: %s\n", item, price)
	}
}

func (db database) price(w http.ResponseWriter, req *http.Request) {
	item := req.URL.Query().Get("item")
	price, ok := db[item]
	if !ok {
		w.WriteHeader(http.StatusNotFound) // 404
		fmt.Fprintf(w, "no such item: %q\n", item)
		return
	}
	fmt.Fprintf(w, "%s\n", price)
}
```
语句http.HandlerFunc(db.list)是一个转换而非一个函数调用，因为http.HandlerFunc是一个类型。
```go
package http

type HandlerFunc func(w ResponseWriter, r *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
  f(w, r)
}
```
这是一个实现了接口http.Handler的方法的函数类型。ServeHTTP方法的行为是调用了它的函数本身。因此HandlerFunc是一个让函数值满足一个接口的适配器，这里函数和这个接口仅有的方法有相同的函数签名。
```go
// ServeMux 提供了一个HandleFunc方法，可以简化为下面的方式
mux.HandleFunc("/list", db.list)
mux.HandleFunc("/price", db.price)
```
net/http包提供了一个全局的ServeMux实例DefaultServerMux和包级别的http.Handle和http.HandleFunc函数。
```go
func main() {
  db := database{"shoes": 50, "socks": 5}
  http.HandleFunc("/list", db.list)
  http.HandleFunc("/price", db.price)
  log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```
`http.HandleFunc`:	将 URL 路径与处理函数绑定到默认路由器（`http.DefaultServeMux`） 
`http.ListenAndServe`:	启动服务器，`nil` 表示使用默认路由器 
`log.Fatal`:	捕获服务器启动错误（如端口冲突）并退出程序 
`w http.ResponseWriter`:	用于向客户端写入 HTTP 响应（如 `fmt.Fprintf(w, ...)`） 
`req *http.Request`:	包含客户端请求信息（如查询参数 `req.URL.Query().Get("item")`） 

#### error接口
error接口定义了错误处理方法，任何实现了error接口的类型都可以作为错误处理对象。
```go
// error接口
type error interface {
  Error() string
}
```
创建一个error最简单的方法就是调用errors.New函数，它会根据传入的错误信息返回一个新的error。
```go
package errors

func New(text string) error { return &errorString{text} }

type errorString struct { text string }

func (e *errorString) Error() string { return e.text }
```
承载errorString的类型是一个结构体而非一个字符串，这是为了保护它表示的错误避免粗心（或有意）的更新。并且因为是指针类型*errorString满足error接口而非errorString类型，所以每个New函数的调用都分配了一个独特的和其他错误不相同的实例。我们也不想要重要的error例如io.EOF和一个刚好有相同错误消息的error比较后相等。

调用errors.New函数是非常稀少的，因为有一个方便的封装函数fmt.Errorf，它还会处理字符串格式化。
```go
package fmt

import "errors"

func Errorf(format string, args ...interface{}) error {
  return errors.New(Sprintf(format, args...))
}
```

### 类型断言
类型断言是一个使用在接口值上的操作。语法：`x.(T)`,x表示一个接口的类型和T表示一个类型。一个类型断言检查它操作对象的动态类型是否和断言的类型匹配。 
**类型断言（Type Assertion）** 是一种在运行时检查接口值的动态类型，并将其转换为具体类型或其他接口类型的机制。

#### 基本语法
**(1) 安全形式（返回布尔值）**
```go
value, ok := interfaceVar.(ConcreteType)
```
如果`interfaceVar`的动态类型是`ConcreteType`：
- `value` 为转换后的值。
- `ok` 为 `true`。

如果类型不匹配：
- `value` 为 `ConcreteType` 的零值。
- `ok` 为 `false`**（不会 panic）**。

**(2) 直接形式（可能 panic）**
```go
value := interfaceVar.(ConcreteType)
```
- 类型匹配时：`value`为转换后的值。
- 类型不匹配时：**触发运行时 panic**。

#### 核心用途
**(1) 从接口中提取具体值**
```go
var i interface{} = "hello"
s, ok := i.(string)  // s = "hello", ok = true
n, ok := i.(int)     // n = 0, ok = false
```
**(2) 判断接口的动态类型**
```go
func printType(v interface{}) {
  switch v := v.(type) {  // 类型开关（Type Switch）
  case int:
      fmt.Println("int:", v)
  case string:
      fmt.Println("string:", v)
  default:
      fmt.Printf("unknown: %T\n", v)
  }
}
```
**(3) 接口转换**
检查一个接口是否实现了另一个接口：
```go
type Reader interface { Read() }
type Writer interface { Write() }

var r Reader = /* ... */
w, ok := r.(Writer)  // 检查是否同时实现了 Writer
```

#### 底层机制
**运行时检查**：类型断言会在运行时检查接口值的动态类型。 
**性能开销**：相比直接方法调用，类型断言有微小性能损耗（需查表）。

#### 基于类型断言区别错误类型
```go
import (
  "errors"
  "syscall"
)

var ErrNotExist = errors.New("file does not exist")

// IsNotExist 返回一个布尔值，指示错误是否已知并报告文件或目录不存在。
// ErrNotExist 以及一些系统调用错误都满足此条件。
func IsNotExist(err error) bool {
  if pe, ok := err.(*PathError); ok {
    err = pe.Err
  }
  return err == syscall.ENOENT || err == ErrNotExist
}
```

#### 通过类型断言询问接口
通过类型断言，我们可以检查一个接口的值是否实现了特定的方法。例如，我们可以检查一个接口的值是否实现了 io.Writer 接口，并调用 WriteString 方法。
```go
// writeString 将 s 写入 w。
// 如果 w 具有 WriteString 方法，则调用该方法而不是 w.Write。
func writeString(w io.Writer, s string) (n int, err error) {
	type stringWriter interface {
		WriteString(string) (n int, err error)
	}
	if sw, ok := w.(stringWriter); ok {
		return sw.WriteString(s) // 避免复制
	}
	return w.Write([]byte(s)) // 分配临时副本
}

func writeHeader(w io.Writer, contentType string) error {
	if _, err := writeString(w, "Content-Type: "); err != nil {
		return err
	}
	if _, err := writeString(w, contentType); err != nil {
		return err
	}
	// ...
}
```

### 类型分支
类型分支是 Go 语言中一种特殊的 switch 语句，用于检查接口值的动态类型。它允许你根据接口变量实际存储的具体类型来执行不同的代码逻辑。
```go
switch v := i.(type) {
case T1:
    // v 的类型是 T1
case T2:
    // v 的类型是 T2
default:
    // 默认情况
}
```

#### 特点
1. 特殊语法：使用 `i.(type)` 作为 switch 表达式
2. 变量声明：可以在 switch 中声明一个新变量（如 `v`）来获取类型断言后的值
3. 类型匹配：每个 case 指定一个类型进行匹配
4. 默认情况：可以使用 default 分支处理未匹配的类型

与普通类型断言的区别: 
| 特性       | 类型分支               | 普通类型断言           |
| ---------- | ---------------------- | ---------------------- |
| 语法       | `switch v := i.(type)` | `v, ok := i.(T)`       |
| 多类型检查 | 支持多个 case          | 一次只能检查一个类型   |
| 失败处理   | 自动跳转到 default     | 需要手动检查 ok 返回值 |
| 变量作用域 | 整个 switch 语句       | 当前作用域             |

## Goroutines和Channels
支持“顺序通信进程”（communicating sequential processes）或被简称为CSP。CSP是一种现代的并发编程模型，在这种编程模型中值会在不同的运行实例（goroutine）中传递，尽管大多数情况下仍然是被限制在单一实例中。

### Goroutines
在Go语言中，每一个并发的执行单元叫作一个goroutine。当一个程序启动时，其主函数即在一个单独的goroutine中运行，我们叫它main goroutine。新的goroutine会用go语句来创建。语法：函数或方法前加go关键字。go语句会使其语句中的函数在一个新创建的goroutine中运行。而go语句本身会迅速地完成。
```go
func main() {
	go spinner(100 * time.Millisecond)
	const n = 45
	fibN := fib(n) // slow
	fmt.Printf("\rFibonacci(%d) = %d\n", n, fibN)
}

func spinner(delay time.Duration) {
	for {
		for _, r := range `-\|/` {
			fmt.Printf("\r%c", r)
			time.Sleep(delay)
		}
	}
}

func fib(x int) int {
	if x < 2 {
		return x
	}
	return fib(x-1) + fib(x-2)
}
```
主函数返回时，所有的goroutine都会被直接打断，程序退出。

### Channels
Channels 是 Go 语言中用于通信的机制。它们允许多个 goroutine 之间进行数据交换。一个channel是一个通信机制，它可以让一个goroutine通过它给另一个goroutine发送值信息。每个channel都有一个特殊的类型，也就是channels可发送数据的类型。
```go
ch := make(chan int) // ch 的类型为 'chan int'
```
和map类似，channel也对应一个make创建的底层数据结构的引用。

一个channel有发送和接受两个主要操作，都是通信行为。一个发送语句将一个值从一个goroutine通过channel发送到另一个执行接收操作的goroutine。发送和接收两个操作都使用<-运算符。在发送语句中，<-运算符分割channel和要发送的值。在接收语句中，<-运算符写在channel对象之前。一个不使用接收结果的接收操作也是合法的。
```go
ch <- x    // 发送语句
x = <-ch   // 赋值语句中的接收表达式
<-ch       // 接收语句；结果被丢弃
```
Channel还支持close操作，用于关闭channel，语法：`close(ch)`，随后对基于该channel的任何发送操作都将导致panic异常。对一个已经被close过的channel进行接收操作依然可以接受到之前已经成功发送的数据；如果channel中已经没有数据的话将产生一个零值的数据。

channel的零值是nil，对一个nil的channel发送和接收操作会永远阻塞，在select语句中操作nil的channel永远都不会被select到。

以最简单方式调用make函数创建的是一个无缓存的channel，但是也可以指定第二个整型参数，对应channel的容量。如果channel的容量大于零，那么该channel就是带缓存的channel。
```go
ch = make(chan int)    // 无缓冲通道
ch = make(chan int, 0) // 无缓冲通道
ch = make(chan int, 3) // 容量为 3 的缓冲通道
```

#### 无缓存Channels
一个基于无缓存Channels的发送操作将导致发送者goroutine阻塞，直到另一个goroutine在相同的Channels上执行接收操作，当发送的值通过Channels成功传输之后，两个goroutine可以继续执行后面的语句。反之，如果接收操作先发生，那么接收者goroutine也将阻塞，直到有另一个goroutine在相同的Channels上执行发送操作。

基于无缓存Channels的发送和接收操作将导致两个goroutine做一次同步操作。因为这个原因，无缓存Channels有时候也被称为同步Channels。当通过一个无缓存Channels发送数据时，接收者收到数据发生在再次唤醒唤醒发送者goroutine之前（happens before）

**消息事件**： 基于channels发送消息来强调通讯发生的时刻。可以用`struct{}`空结构体作为channels元素的类型，虽然也可以使用bool或int类型实现同样的功能，`done <- 1`语句也比`done <- struct{}{}`更短。

#### 串联的Channels（Pipeline）
Channels也可以用于将多个goroutine连接在一起，一个Channel的输出作为下一个Channel的输入。这种串联的Channels就是所谓的管道（pipeline）。
```go
x, ok = <-ch  // ok 为 true 表示 ch 有数据可读, 为 false 表示 ch 已关闭并且里面没有值可接收。
```

#### 单方向的Channel
单方向的Channel只能用于发送或接收数据，不能同时发送和接收数据。
```go
func counter(out chan<- int){/** */} // out 是一个只发送数据的Channel
func counter(in <-chan int){/** */}  // in 是一个只接收数据的Channel
```
任何双向channel向单向channel变量的赋值操作都将导致该隐式转换。这里并没有反向转换的语法：也就是不能将一个类似chan<- int类型的单向型的channel转换为chan int类型的双向型的channel。

#### 带缓存的Channels
带缓存的Channel内部持有一个元素队列。队列的最大容量是在调用make函数创建channel时通过第二个参数指定的。
```go
ch = make(chan int, 3) // 容量为 3 的缓冲通道，可以持有3个int元素

cap(ch) // cap函数可以获取channel的容量。返回 3

ch <- 2
len(ch) /// len函数可以获取channel的长度。返回 1
```
向缓存Channel的发送操作就是向内部缓存队列的尾部插入元素，接收操作则是从队列的头部删除元素。如果内部缓存队列是满的，那么发送操作将阻塞直到因另一个goroutine执行接收操作而释放了新的队列空间。相反，如果channel是空的，接收操作将阻塞直到有另一个goroutine执行发送操作而向队列插入元素。

如果使用了无缓存的channel，慢的goroutines没有人接收就会被永远卡住。这种情况，称为goroutines泄漏，这将是一个BUG。和垃圾变量不同，泄漏的goroutines并不会被自动回收，因此确保每个不再需要的goroutine能正常退出是重要的。

### sync.WaitGroup
`sync.WaitGroup` 是一种用于 **等待一组 Goroutine 完成** 的同步原语。它特别适用于需要阻塞主线程（或父 Goroutine）直到所有子 Goroutine 执行完毕的场景。

#### 核心功能
**计数器管理**：WaitGroup 内部维护一个计数器，通过 `Add()` 增加计数，`Done()` 减少计数，`Wait()` 阻塞直到计数器归零。 
**典型用途**：等待多个并发任务（如 HTTP 请求、文件处理）全部完成后再继续主流程。

#### 关键方法
| 方法                                  | 作用                                             |
| ------------------------------------- | ------------------------------------------------ |
| `func (wg *WaitGroup) Add(delta int)` | 增加计数器值（`delta` 可为负数，但通常为正数）。 |
| `func (wg *WaitGroup) Done()`         | 计数器减 1（等价于 `Add(-1)`）。                 |
| `func (wg *WaitGroup) Wait()`         | 阻塞当前 Goroutine，直到计数器归零。             |

示例:
```go
func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done() // 确保任务完成时计数器减1

	fmt.Printf("Worker %d starting\n", id)
	time.Sleep(time.Second) // 模拟耗时任务
	fmt.Printf("Worker %d done\n", id)
}

func main() {
	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {
		wg.Add(1) // 每个任务开始前增加计数
		go worker(i, &wg)
	}

	wg.Wait() // 阻塞直到所有任务完成
	fmt.Println("All workers completed")
}
```

#### 注意事项
**(1) Add 必须在 Goroutine 外调用**
```go
go func() {
  wg.Add(1) // 可能来不及执行，导致 Wait 提前退出
  defer wg.Done()
  // ...
}()
wg.Wait()
```
**(2) 避免计数器为负** 调用 `Done()` 次数超过 `Add()` 的累计值会引发 panic：
```go
wg.Add(2)
wg.Done()
wg.Done()
wg.Done() // panic: sync: negative WaitGroup counter
```
**(3) 不要复制 `WaitGroup`** `WaitGroup` 是值类型，但复制后会导致状态不一致：
```go
var wg1 sync.WaitGroup
wg1.Add(1)
wg2 := wg1 // 错误！应始终传递指针
go func() { defer wg2.Done() }()
wg1.Wait() // 可能无法正确等待
```
正确做法：始终传递 `WaitGroup` 的指针（如 `func(wg *sync.WaitGroup)`）。

### 并发的退出
广播机制：不要向channel发送值，而是用关闭一个channel来进行广播。
```go
var done = make(chan struct{})

// 被调用的时候会轮询退出状态。
func cancelled() bool {
  for {
    select {
    case <-done:
        // 排空fileSizes channel以允许现有的 goroutines 完成。
        for range fileSizes {
          // Do nothing.
        }
        return true
    default:
        return false
    }
  }
}

// 当检测到输入时取消遍历。
go func() {
  os.Stdin.Read(make([]byte, 1)) // 读取单个字节
  close(done)
}()
```

### 竞争条件
一个函数在线性程序中可以正确地工作。如果在并发的情况下，这个函数依然可以正确地工作的话，那么这个函数是并发安全的，并发安全的函数不需要额外的同步工作。对于某个类型来说，如果其所有可访问的方法和操作都是并发安全的话，那么该类型便是并发安全的。

竞争条件指的是程序在多个goroutine交叉执行操作时，没有给出正确的结果。

#### 数据竞争
数据竞争会在两个以上的goroutine并发访问相同的变量且至少其中一个为写操作时发生。

解决办法： 
1. 不要去写变量。直接在初始化阶段就对变量赋值并不再修改。
2. 避免从多个goroutine访问变量。把变量限定在了一个单独的goroutine中。 
由于其它的goroutine不能够直接访问变量，它们只能使用一个channel来发送请求给指定的goroutine来查询更新变量。这也就是Go的口头禅“不要使用共享数据来通信；使用通信来共享数据”。一个提供对一个指定的变量通过channel来请求的goroutine叫做这个变量的monitor（监控）goroutine。
```go
// 包银行提供了一个具有一个账户的并发安全银行。
package bank

var deposits = make(chan int) // 发送金额至存款
var balances = make(chan int) // 接收余额

func Deposit(amount int) { deposits <- amount }
func Balance() int       { return <-balances }

func teller() {
  var balance int // 余额仅限于teller goroutine
  for {
    select {
      case amount := <-deposits:
          balance += amount
      case balances <- balance:
      }
  }
}

func init() {
  go teller() // 启动监控goroutine
}
```
3. 使用互斥锁。允许很多goroutine去访问变量，但是在同一个时刻最多只有一个goroutine在访问。

### sync.Mutex互斥锁
用一个容量只有1的channel来保证最多只有一个goroutine在同一时刻访问一个共享变量。一个只能为1和0的信号量叫做二元信号量（binary semaphore）。
``` go
import "sync"

var (
	mu      sync.Mutex // guards balance
	balance int
)

func Deposit(amount int) {
	mu.Lock()
	balance = balance + amount
	mu.Unlock()
}

func Balance() int {
	mu.Lock()
	b := balance
	mu.Unlock()
	return b
}

// 每次一个goroutine访问bank变量时（这里只有balance余额变量），它都会调用mutex的Lock方法来获取一个互斥锁。如果其它的goroutine已经获得了这个锁的话，这个操作会被阻塞直到其它goroutine调用了Unlock使该锁变回可用状态。mutex会保护共享变量。惯例来说，被mutex所保护的变量是在mutex变量声明之后立刻声明的。如果你的做法和惯例不符，确保在文档里对你的做法进行说明。
```
在Lock和Unlock之间的代码段中的内容goroutine可以随便读取或者修改，这个代码段叫做临界区。锁的持有者在其他goroutine获取该锁之前需要调用Unlock。goroutine在结束后释放锁是必要的，无论以哪条路径通过函数都需要释放，即使是在错误路径中，也要记得释放。
> 没法对一个已经锁上的mutex来再次上锁——这会导致程序死锁

### sync.RWMutex读写锁
`sync.RWMutex`允许多个只读操作并行执行，但写操作会完全互斥。这种锁叫作“多读单写”锁（multiple readers, single writer lock）
```go
var mu sync.RWMutex
var balance int
func Balance() int {
  mu.RLock() // readers lock
  defer mu.RUnlock()
  return balance
}
```
RWMutex只有当获得锁的大部分goroutine都是读操作，而锁在竞争条件下，也就是说，goroutine们必须等待才能获取到锁的时候，RWMutex才是最能带来好处的。RWMutex需要更复杂的内部记录，所以会让它比一般的无竞争锁的mutex慢一些。

### sync.Once惰性初始化
如果初始化成本比较大的话，那么将初始化延迟到需要的时候再去做就是一个比较好的选择。如果在程序启动的时候就去做这类初始化的话，会增加程序的启动时间，并且因为执行的时候可能也并不需要这些变量，所以实际上有一些浪费。
```go
var loadIconsOnce sync.Once
var icons map[string]image.Image
// Concurrency-safe.
func Icon(name string) image.Image {
  loadIconsOnce.Do(loadIcons)
  return icons[name]
}
```

### 竞争条件检测
在go build，go run或者go test命令后面加上-race的flag，就会使编译器创建一个你的应用的“修改”版或者一个附带了能够记录所有运行期对共享变量访问工具的test，并且会记录下每一个读或者写共享变量的goroutine的身份信息。另外，修改版的程序会记录下所有的同步事件，比如go语句，channel操作，以及对(*sync.Mutex).Lock，(*sync.WaitGroup).Wait等等的调用。

竞争检查器会检查这些事件，会寻找在哪一个goroutine中出现了这样的case，例如其读或者写了一个共享变量，这个共享变量是被另一个goroutine在没有进行干预同步操作便直接写入的。这种情况也就表明了是对一个共享变量的并发访问，即数据竞争。这个工具会打印一份报告，内容包含变量身份，读取和写入的goroutine中活跃的函数的调用栈。这些信息在定位问题时通常很有用。

### Goroutines和线程

#### 动态栈
每一个OS线程都有一个固定大小的内存块（一般会是2MB）来做栈，这个栈会用来存储当前正在被调用或挂起（指在调用其它函数时）的函数的内部变量。对于go程序来说，同时创建成百上千个goroutine是非常普遍的，如果每一个goroutine都需要这么大的栈的话，那这么多的goroutine就不太可能了。固定大小的栈对于更复杂或者更深层次的递归函数调用来说显然是不够的。修改固定的大小可以提升空间的利用率，允许创建更多的线程，并且可以允许更深的递归调用，不过这两者是没法同时兼备的。

一个goroutine会以一个很小的栈开始其生命周期，一般只需要2KB。一个goroutine的栈，和操作系统线程一样，会保存其活跃或挂起的函数调用的本地变量，但是和OS线程不太一样的是，一个goroutine的栈大小并不是固定的；栈的大小会根据需要动态地伸缩。而goroutine的栈的最大值有1GB。

####  Goroutine调度
OS线程会被操作系统内核调度。每几毫秒，一个硬件计时器会中断处理器，这会调用一个叫作scheduler的内核函数。这个函数会挂起当前执行的线程并将它的寄存器内容保存到内存中，检查线程列表并决定下一次哪个线程可以被运行，并从内存中恢复该线程的寄存器信息，然后恢复执行该线程的现场并开始执行线程。因为操作系统线程是被内核所调度，所以从一个线程向另一个“移动”需要完整的上下文切换，也就是说，保存一个用户线程的状态到内存，恢复另一个线程的到寄存器，然后更新调度器的数据结构。这几步操作很慢，因为其局部性很差需要几次内存访问，并且会增加运行的cpu周期。

Go的运行时包含了其自己的调度器，这个调度器使用了一些技术手段，比如m:n调度，因为其会在n个操作系统线程上多工（调度）m个goroutine。Go调度器的工作和内核的调度是相似的，但是这个调度器只关注单独的Go程序中的goroutine（按程序独立）。

和操作系统的线程调度不同的是，Go调度器并不是用一个硬件定时器，而是被Go语言“建筑”本身进行调度的。例如当一个goroutine调用了time.Sleep，或者被channel调用或者mutex操作阻塞时，调度器会使其进入休眠并开始执行另一个goroutine，直到时机到了再去唤醒第一个goroutine。因为这种调度方式不需要进入内核的上下文，所以重新调度一个goroutine比调度一个线程代价要低得多。

#### GOMAXPROCS
Go的调度器使用了一个叫做GOMAXPROCS的变量来决定会有多少个操作系统的线程同时执行Go的代码。其默认的值是运行机器上的CPU的核心数，所以在一个有8个核心的机器上时，调度器一次会在8个OS线程上去调度GO代码。（GOMAXPROCS是前面说的m:n调度中的n）。

可以用GOMAXPROCS的环境变量来显式地控制这个参数，或者也可以在运行时用runtime.GOMAXPROCS函数来修改它。
```go
for {
  go fmt.Print(0)
  fmt.Print(1)
}

$ GOMAXPROCS=1 go run hacker-cliché.go
// 111111111111111111110000000000000000000011111...

$ GOMAXPROCS=2 go run hacker-cliché.go
// 010101010101010101011001100101011010010100110...
```

#### Goroutine没有ID号
goroutine没有可以被程序员获取到的身份（id）的概念。这一点是设计上故意而为之，由于thread-local storage总是会被滥用。比如说，一个web server是用一种支持tls的语言实现的，而非常普遍的是很多函数会去寻找HTTP请求的信息，这代表它们就是去其存储层（这个存储层有可能是tls）查找的。这就像是那些过分依赖全局变量的程序一样，会导致一种非健康的“距离外行为”，在这种行为下，一个函数的行为可能并不仅由自己的参数所决定，而是由其所运行在的线程所决定。因此，如果线程本身的身份会改变——比如一些worker线程之类的——那么函数的行为就会变得神秘莫测。

Go鼓励更为简单的模式，这种模式下参数（外部显式参数和内部显式参数。tls 中的内容算是"外部"隐式参数）对函数的影响都是显式的。这样不仅使程序变得更易读，而且会让我们自由地向一些给定的函数分配子任务时不用担心其身份信息影响行为。

## 包和工具
Go语言有超过100个的标准包（可以用go list std | wc -l命令查看标准包的具体数目），标准库为大多数的程序提供了必要的基础构件。在Go的社区，有很多成熟的包被设计、共享、重用和改进，目前互联网上已经发布了非常多的Go语言开源包，它们可以通过 [http://godoc.org](http://godoc.org) 检索。

Go还自带了工具箱，里面有很多用来简化工作区和包管理的小工具。

### 包简介
任何包系统设计的目的都是为了简化大型程序的设计和维护工作，通过将一组相关的特性放进一个独立的单元以便于理解和更新，在每个单元更新的同时保持和程序中其它单元的相对独立性。这种模块化的特性允许每个包可以被其它的不同项目共享和重用，在项目范围内、甚至全球范围统一的分发和复用。

每个包一般都定义了一个不同的名字空间用于它内部的每个标识符的访问。每个名字空间关联到一个特定的包，让我们给类型、函数等选择简短明了的名字，这样可以在使用它们的时候减少和其它部分名字的冲突。

每个包还通过控制包内名字的可见性和是否导出来实现封装特性。通过限制包成员的可见性并隐藏包API的具体实现，将允许包的维护者在不影响外部包用户的前提下调整包的内部实现。通过限制包内变量的可见性，还可以强制用户通过某些特定函数来访问和更新内部变量，这样可以保证内部变量的一致性和并发时的互斥约束。

当我们修改了一个源文件，我们必须重新编译该源文件对应的包和所有依赖该包的其他包。即使是从头构建，Go语言编译器的编译速度也明显快于其它编译语言。Go语言的闪电般的编译速度主要得益于三个语言特性。第一点，所有导入的包必须在每个文件的开头显式声明，这样的话编译器就没有必要读取和分析整个源文件来判断包的依赖关系。第二点，禁止包的环状依赖，因为没有循环依赖，包的依赖关系形成一个有向无环图，每个包可以被独立编译，而且很可能是被并发编译。第三点，编译后包的目标文件不仅仅记录包本身的导出信息，目标文件同时还记录了包的依赖关系。因此，在编译一个包的时候，编译器只需要读取每个直接导入包的目标文件，而不需要遍历所有依赖的的文件（很多都是重复的间接依赖）。

### 导入路径
每个包是由一个全局唯一的字符串所标识的导入路径定位。出现在import语句中的导入路径也是字符串。
```go
import (
  "fmt"
  "math/rand"
  "encoding/json"

  "golang.org/x/net/html"

  "github.com/go-sql-driver/mysql"
)
```
Go语言的规范并没有指明包的导入路径字符串的具体含义，导入路径的具体含义是由构建工具来解释的。

如果计划分享或发布包，那么导入路径最好是全球唯一的。为了避免冲突，所有非标准库包的导入路径建议以所在组织的互联网域名为前缀；而且这样也有利于包的检索。例如，上面的import语句导入了Go团队维护的HTML解析器和一个流行的第三方维护的MySQL驱动。

### 包声明
在每个Go语言源文件的开头都必须有包声明语句。包声明语句的主要目的是确定当前包被其它包导入时默认的标识符（也称为包名）。

例如，math/rand包的每个源文件的开头都包含package rand包声明语句，所以当你导入这个包，你就可以用rand.Int、rand.Float64类似的方式访问包的成员。

通常来说，默认的包名就是包导入路径名的最后一段，因此即使两个包的导入路径不同，它们依然可能有一个相同的包名。例如，math/rand包和crypto/rand包的包名都是rand。

关于默认包名一般采用导入路径名的最后一段的约定也有三种例外情况: 
1. 包对应一个可执行程序，也就是main包，这时候main包本身的导入路径是无关紧要的。名字为main的包是给go build构建命令一个信息，这个包编译完之后必须调用连接器生成一个可执行程序。
2. 包所在的目录中可能有一些文件名是以_test.go为后缀的Go源文件（前面必须有其它的字符，因为以_或.开头的源文件会被构建工具忽略），并且这些源文件声明的包名也是以_test为后缀名的。这种目录可以包含两种包：一种是普通包，另一种则是测试的外部扩展包。所有以_test为后缀包名的测试外部扩展包都由go test命令独立编译，普通包和测试的外部扩展包是相互独立的。测试的外部扩展包一般用来避免测试代码中的循环导入依赖。
3. 一些依赖版本号的管理工具会在导入路径后追加版本号信息，例如“gopkg.in/yaml.v2”。这种情况下包的名字并不包含版本号后缀，而是yaml。

### 导入声明
可以在一个Go语言源文件包声明语句之后，其它非导入声明语句之前，包含零到多个导入包声明语句。每个导入声明可以单独指定一个导入路径，也可以通过圆括号同时导入多个导入路径
```go
import "fmt"
import "os"

// 等价于，但是更常用
import (
  "fmt"
  "os"
)
```
导入的包之间可以通过添加空行来分组；通常将来自不同组织的包独自分组。包的导入顺序无关紧要，但是在每个分组中一般会根据字符串顺序排列。（gofmt和goimports工具都可以将不同分组导入的包独立排序。）
```go
import (
	"fmt"
	"html/template"
	"os"

	"golang.org/x/net/html"
	"golang.org/x/net/ipv4"
)
```
如果想同时导入两个有着名字相同的包，例如math/rand包和crypto/rand包，那么导入声明必须至少为一个同名包指定一个新的包名以避免冲突。这叫做导入包的重命名。
```go
import (
  "crypto/rand"
  mrand "math/rand" // 替代名称 mrand 避免冲突
)
```
导入包的重命名只影响当前的源文件。其它的源文件如果导入了相同的包，可以用导入包原本默认的名字或重命名为另一个完全不同的名字。

导入包重命名是一个有用的特性，它不仅仅只是为了解决名字冲突。如果导入的一个包名很笨重，特别是在一些自动生成的代码中，这时候用一个简短名称会更方便。选择用简短名称重命名导入包时候最好统一，以避免包名混乱。选择另一个包名称还可以帮助避免和本地普通变量名产生冲突。例如，如果文件中已经有了一个名为path的变量，那么我们可以将“path”标准包重命名为pathpkg。

每个导入声明语句都明确指定了当前包和被导入包之间的依赖关系。如果遇到包循环导入的情况，Go语言的构建工具将报告错误。

### 包的匿名导入
如果只是导入一个包而并不使用导入的包将会导致一个编译错误。但是有时候只是想利用导入包而产生的副作用：它会计算包级变量的初始化表达式和执行导入包的init初始化函数。这时候需要抑制“unused import”编译错误，可以用下划线_来重命名导入的包。下划线_为空白标识符，并不能被访问。
```go
import _ "image/png" // 注册 PNG 解码器
```
这个被称为包的匿名导入。它通常是用来实现一个编译时机制，然后通过在main主程序入口选择性地导入附加的包。

image/png 包的初始化函数会注册 PNG 解码器。
```go
package png // image/png

func Decode(r io.Reader) (image.Image, error)
func DecodeConfig(r io.Reader) (image.Config, error)

func init() {
  const pngHeader = "\x89PNG\r\n\x1a\n"
  image.RegisterFormat("png", pngHeader, Decode, DecodeConfig)
}
```

### 包的命名
要求包的命名短小有描述性且无歧义。要尽量避免包名使用可能被经常用于局部变量的名字，这样可能导致用户重命名导入包，例如前面看到的path包。 
包名一般采用单数的形式。 
要避免包名有其它的含义。 
当设计一个包的时候，需要考虑包名和成员名两个部分如何很好地配合。

### 工具
Go语言的工具箱集合了一系列功能的命令集。它可以看作是一个包管理器（类似于Linux中的apt和rpm工具），用于包的查询、计算包的依赖关系、从远程版本控制系统下载它们等任务。
它也是一个构建系统，计算文件的依赖关系，然后调用编译器、汇编器和链接器构建程序，虽然它故意被设计成没有标准的make命令那么复杂。它也是一个单元测试和基准测试的驱动程序。

#### 工作区结构
对于大多数的Go语言用户，只需要配置一个名叫GOPATH的环境变量，用来指定当前工作目录即可。当需要切换到不同工作区的时候，只要更新GOPATH就可以了。
```bash
$ export GOPATH=$HOME/gobook
$ go get gopl.io/...
```
GOROOT用来指定Go的安装目录，还有它自带的标准库包的位置。GOROOT的目录结构和GOPATH类似，因此存放fmt包的源代码对应目录应该为$GOROOT/src/fmt。一般不需要设置GOROOT，默认情况下Go语言安装工具会将其设置为安装的目录路径。

`go env`命令用于查看Go语言工具涉及的所有环境变量的值，包括未设置环境变量的默认值。

#### 下载包
使用命令`go get`可以下载一个单一的包或者用...下载整个子目录里面的每个包。Go语言工具箱的go命令同时计算并下载所依赖的每个包。

`go get`命令支持当前流行的托管网站GitHub、Bitbucket和Launchpad，可以直接向它们的版本控制系统请求代码。

版本控制：
- `@latest`：最新版本
- `@v1.2.3`：指定版本
- `@commit-hash`：特定提交

##### `go install`
`go install`: 编译并安装包或程序到 $GOPATH/bin

##### `go mod`
`go mod`命令 
- `go mod init`: 初始化新模块
- `go mod tidy`: 添加缺失和移除无用依赖
- `go mod vendor`: 创建vendor目录
- `go mod download`: 下载模块到本地缓存

#### 构建包
`go build`命令编译命令行参数指定的每个包。如果包是一个库，则忽略输出结果；这可以用于检测包是可以正确编译的。如果包的名字是main，`go build`将调用链接器在当前目录创建一个可执行程序；以导入路径的最后一段作为可执行程序的名字。

常用选项: 
- `-o`: 指定输出文件名。
- `-v`：显示编译的包名
- `-race`：启用竞态检测

##### `go run`
`go run`命令：直接编译并运行 Go 程序。不会生成可执行文件；适合快速测试代码；支持多文件：`go run *.go`

##### `go test`
`go test`: 运行测试
常用标志：
- `-v`：详细输出
- `-cover`：覆盖率分析
- `-race`：竞态检测
- `-bench`：运行基准测试 
- `-run`: 运行匹配正则表达式的测试
- `-coverprofile`: 生成覆盖文件
测试缓存：使用 `-count=1` 禁用缓存

##### `go vet`
`go vet`: 检查代码中的常见错误
检查内容：错误的printf格式；错误的锁使用；其他可疑结构。

##### `go fmt`
`go fmt`: 格式化代码为标准风格 
等价于：`gofmt -w -l .`

##### `go generate`
`go generate`: 执行代码生成指令
```bash
//go:generate protoc --go_out=. protofile.proto
```
特点：通过特殊注释标记；需手动运行`go generate`

##### `go tool`
`go tool`: 访问Go工具
重要子工具：
- `pprof`：性能分析
- `trace`：执行跟踪
- `compile`：查看编译器优化
- `nm`：查看符号表
- `cover`: 查看代码覆盖

#### 包文档
Go语言的编码风格鼓励为每个包提供良好的文档。包中每个导出的成员和包声明前都应该包含目的和用法说明的注释。

Go语言中的文档注释一般是完整的句子，第一行通常是摘要说明，以被注释者的名字开头。注释中函数的参数或其它的标识符并不需要额外的引号或其它标记注明。例如，下面是fmt.Fprintf的文档注释。
```go
// Fprintf formats according to a format specifier and writes to w.
// It returns the number of bytes written and any write error encountered.
func Fprintf(w io.Writer, format string, a ...interface{}) (int, error)
```
Fprintf函数格式化的细节在fmt包文档中描述。如果注释后紧跟着包声明语句，那注释对应整个包的文档。包文档对应的注释只能有一个（如果有多个，它们会组合成一个包文档注释），包注释可以出现在任何一个源文件中。如果包的注释内容比较长，一般会放到一个独立的源文件中；fmt包注释就有300行之多。这个专门用于保存包文档的源文件通常叫doc.go。

`go doc`命令，该命令打印其后所指定的实体的声明与文档注释，该实体可以是一个包、具体的包成员、一个方法或一个类型。`go doc -src pkg` 查看源代码

**godoc工具** 提供可以相互交叉引用的HTML页面，但是包含和`go doc`命令相同以及更多的信息。
```bash
$ godoc -http :8000  #运行godoc服务，在http://localhost:8000/pkg 查看文档
```

### 内部包
在Go语言程序中，包是最重要的封装机制。没有导出的标识符只在同一个包内部可以访问，而导出的标识符则是面向全宇宙都是可见的。

Go语言的构建工具对包含internal名字的路径段的包导入路径做了特殊处理。这种包叫internal包，一个internal包只能被和internal目录有同一个父目录的包所导入。例如，net/http/internal/chunked内部包只能被net/http/httputil或net/http包导入，但是不能被net/url包导入。不过net/url包却可以导入net/http/httputil包。
```go
"net/http"
"net/http/internal/chunked"
"net/http/httputil"
"net/url"
```

#### 查询包
`go list`命令可以查询可用包的信息。其最简单的形式，可以测试包是否在工作区并打印它的导入路径：
```bash
$ go list github.com/go-sql-driver/mysql
github.com/go-sql-driver/mysql
```
`go list`命令的参数还可以用"..."表示匹配任意的包的导入路径。
```bash
$ go list ...   # 列出工作区中的所有包
# archive/zip
# bufio
# bytes

$ go list gopl.io/ch3/...   # 列出特定目录下的所有包
# gopl.io/ch3/basename1
# gopl.io/ch3/basename2

$ go list ...xml...   # 列出和某个主题相关的所有包
# encoding/xml
# gopl.io/ch7/xmlselect
```
常用选项：
- `-f`：自定义格式输出
- `-m`：列出模块而非包
- `-json`：JSON格式输出

#### 版本与清理

##### `go version`
`go version`: 显示Go或二进制文件的版本信息

##### `go clean`
`go clean`: 清理构建缓存 
常用选项：
- `-i`：删除安装的包
- `-cache`：清理构建缓存
- `-testcache`：清理测试缓存

### 包应用介绍

#### fmt包

##### fmt.Printf
`fmt.Printf`:  Go 语言中用于格式化输出的重要函数。
```go
func Printf(format string, a ...interface{}) (n int, err error)
```
- `format`：格式化字符串，包含普通文本和格式化动词（verbs）
- `a ...interface{}`：可变参数，用于替换格式化字符串中的占位符

常用格式化动词：
| 动词 | 说明                     | 示例                  |
|------|--------------------------|-----------------------|
| %v   | 默认格式的值             | Printf("%v", data)    |
| %+v  | 结构体字段名+值          | Printf("%+v", struct) |
| %#v  | Go语法表示的值           | Printf("%#v", data)   |
| %T   | 值类型的Go语法表示       | Printf("%T", data)    |
| %%   | 百分号字面量             | Printf("%%")          |
| %t   | 布尔值（true/false）     | Printf("%t", true)    |
| %d   | 十进制整数               | Printf("%d", 123)     |
| %b   | 二进制表示               | Printf("%b", 5)       |
| %o   | 八进制表示               | Printf("%o", 8)       |
| %x   | 十六进制（小写字母）     | Printf("%x", 255)     |
| %X   | 十六进制（大写字母）     | Printf("%X", 255)     |
| %f   | 浮点数                   | Printf("%f", 3.14)    |
| %e   | 科学计数法（小写e）      | Printf("%e", 1000.0)  |
| %E   | 科学计数法（大写E）      | Printf("%E", 1000.0)  |
| %s   | 字符串                   | Printf("%s", "hello") |
| %q   | 带双引号的字符串         | Printf("%q", "hello") |
| %p   | 指针地址（十六进制表示） | Printf("%p", &x)      |

可以在动词前添加宽度和精度：
```go
fmt.Printf("%10s", "hello")   // 右对齐，宽度10
fmt.Printf("%-10s", "hello")  // 左对齐，宽度10
fmt.Printf("%.2f", 3.14159)   // 浮点数，保留2位小数
fmt.Printf("%10.2f", 3.14159) // 宽度10，保留2位小数
```
> 格式化字符串中的动词数量必须与参数数量匹配
> 可以使用索引指定参数位置，如 %[2]d 表示使用第二个参数
> 对于复杂格式化需求，可以考虑使用 fmt.Sprintf 先构建字符串再输出

##### fmt.Fprintf
`fmt.Fprintf` 是 Go 语言标准库 fmt 包中提供的一个格式化输出函数，它允许将格式化的字符串写入到指定的`io.Writer`接口中，而不是像`fmt.Printf`那样直接输出到标准输出。
```go
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
```
参数说明：
- `w`：实现了`io.Writer`接口的对象（如文件、网络连接、缓冲区等）
- `format`：格式化字符串，包含普通文本和格式化动词（verbs）
- `a ...interface{}`：可变参数，用于替换格式化字符串中的占位符

返回值：
- `n`：写入的字节数
- `err`：写入过程中遇到的错误

fmt.Printf 实际上是 fmt.Fprintf 的一个特例，它固定将输出写入标准输出（os.Stdout）：
```go
func Printf(format string, a ...interface{}) (n int, err error) {
  return Fprintf(os.Stdout, format, a...)
}
```

`fmt.Fprintf` 适用于以下场景：
1. 将格式化内容写入文件
2. 将格式化内容写入网络连接
3. 将格式化内容写入字符串缓冲区
4. 任何需要自定义输出目标的情况

## 测试

### go test
go test命令是一个按照一定的约定和组织来测试代码的程序。在包目录内，所有以`_test.go`为后缀名的源文件在执行go build时不会被构建成包的一部分，它们是go test测试的一部分。

在`*_test.go`文件中，有三种类型的函数：测试函数、基准测试（benchmark）函数、示例函数。一个测试函数是以Test为函数名前缀的函数，用于测试程序的一些逻辑行为是否正确；go test命令会调用这些测试函数并报告测试结果是PASS或FAIL。基准测试函数是以Benchmark为函数名前缀的函数，它们用于衡量一些函数的性能；go test命令会多次运行基准测试函数以计算一个平均的执行时间。示例函数是以Example为函数名前缀的函数，提供一个由编译器保证正确性的示例文档。

### 测试函数
每个测试函数必须导入testing包。测试函数有如下的签名：
```go
func TestName(t *testing.T) {
  // ...
}
```
测试函数的名字必须以Test开头，可选的后缀名必须以大写字母开头：
```go
func TestSin(t *testing.T) { /* ... */ }
func TestCos(t *testing.T) { /* ... */ }
func TestLog(t *testing.T) { /* ... */ }
```
其中t参数用于报告测试失败和附加的日志信息。

```go
// 表格驱动的测试
func TestIsPalindrome(t *testing.T) {
	var tests = []struct {
		input string
		want  bool
	}{
		{"", true},
		{"a", true},
		{"aa", true},
		{"ab", false},
		{"kayak", true},
		{"Evil I did dwell; lewd did I live.", true},
		{"Able was I ere I saw Elba", true},
		{"été", true},
		{"Et se resservir, ivresse reste.", true},
		{"palindrome", false}, // non-palindrome
		{"desserts", false},   // semi-palindrome
	}
	for _, test := range tests {
		if got := IsPalindrome(test.input); got != test.want {
			t.Errorf("IsPalindrome(%q) = %v", test.input, got)
		}
	}
}
```

#### 随机测试
随机测试，通过构造更广泛的随机输入来测试探索函数的行为。

对于一个随机的输入，要知道希望的输出结果有两种处理策略。第一个是编写另一个对照函数，使用简单和清晰的算法，虽然效率较低但是行为和要测试的函数是一致的，然后针对相同的随机输入检查两者的输出结果。第二种是生成的随机输入的数据遵循特定的模式，这样就可以知道期望的输出的模式。

虽然随机测试会有不确定因素，但是它也是至关重要的，我们可以从失败测试的日志获取足够的信息。

#### 白盒测试
基于测试者是否需要了解被测试对象的内部工作原理的分类方法，黑盒测试只需要测试包公开的文档和API行为，内部实现对测试代码是透明的。白盒测试有访问包内部函数和数据结构的权限，因此可以做到一些普通客户端无法实现的测试。

黑盒和白盒这两种测试方法是互补的。黑盒测试一般更健壮，随着软件实现的完善测试代码很少需要更新。它们可以帮助测试者了解真实客户的需求，也可以帮助发现API设计的一些不足之处。相反，白盒测试则可以对内部一些棘手的实现提供更多的测试覆盖。

#### 测试覆盖率
对待测程序执行的测试的程度称为测试的覆盖率。测试覆盖率并不能量化——即使最简单的程序的动态也是难以精确测量的——但是有启发式方法来帮助我们编写有效的测试代码。

这些启发式方法中，语句的覆盖率是最简单和最广泛使用的。语句的覆盖率是指在测试中至少被运行一次的代码占总代码数的比例。

### 基准测试
基准测试是测量一个程序在固定工作负载下的性能。在Go语言中，基准测试函数和普通测试函数写法类似，但是以Benchmark为前缀名，并且带有一个*testing.B类型的参数；*testing.B参数除了提供和*testing.T类似的方法，还有额外一些和性能测量相关的方法。它还提供了一个整数N，用于指定操作执行的循环次数。

下面是IsPalindrome函数的基准测试，其中循环将执行N次。
```go
import "testing"

func BenchmarkIsPalindrome(b *testing.B) {
  for i := 0; i < b.N; i++ {
    IsPalindrome("A man, a plan, a canal: Panama")
  }
}
```

默认情况下不运行任何基准测试。需要通过-bench命令行标志参数手工指定要运行的基准测试函数。该参数是一个正则表达式，用于匹配要执行的基准测试函数的名字，默认值是空的。其中“.”模式将可以匹配所有基准测试函数，但因为这里只有一个基准测试函数，因此和`-bench=IsPalindrome参`数是等价的效果。
```bash
$ go test -bench=.
PASS
BenchmarkIsPalindrome-8 1000000                1035 ns/op
ok      gopl.io/ch11/word2      2.179s
```
结果中基准测试名的数字后缀部分，这里是8，表示运行时对应的GOMAXPROCS的值，这对于一些与并发相关的基准测试是重要的信息。

报告显示每次调用IsPalindrome函数花费1.035微秒，是执行1,000,000次的平均时间。因为基准测试驱动器开始时并不知道每个基准测试函数运行所花的时间，它会尝试在真正运行基准测试前先尝试用较小的N运行测试来估算基准测试函数所需要的时间，然后推断一个较大的时间保证稳定的测量结果。

循环在基准测试函数内实现，而不是放在基准测试框架内实现，这样可以让每个基准测试函数有机会在循环启动前执行初始化代码，这样并不会显著影响每次迭代的平均运行时间。如果还是担心初始化代码部分对测量时间带来干扰，那么可以通过testing.B参数提供的方法来临时关闭或重置计时器，不过这些一般很少会用到。

快的程序往往是伴随着较少的内存分配。`-benchmem`命令行标志参数将在报告中包含内存的分配数据统计。

通过函数参数来指定输入的大小，但是参数变量对于每个具体的基准测试都是固定的。要避免直接修改b.N来控制输入的大小。除非你将它作为一个固定大小的迭代计算输入，否则基准测试的结果将毫无意义。

#### 剖析
当想仔细观察程序的运行速度的时候，最好的方法是性能剖析。剖析技术是基于程序执行期间一些自动抽样，然后在收尾时进行推断；最后产生的统计结果就称为剖析数据。

Go语言支持多种类型的剖析性能分析，每一种关注不同的方面，但它们都涉及到每个采样记录的感兴趣的一系列事件消息，每个事件都包含函数调用时函数调用堆栈的信息。内建的go test工具对几种分析方式都提供了支持。

CPU剖析数据标识了最耗CPU时间的函数。在每个CPU上运行的线程在每隔几毫秒都会遇到操作系统的中断事件，每次中断时都会记录一个剖析数据然后恢复正常的运行。

堆剖析则标识了最耗内存的语句。剖析库会记录调用内部内存分配的操作，平均每512KB的内存申请会触发一个剖析数据。

阻塞剖析则记录阻塞goroutine最久的操作，例如系统调用、管道发送和接收，还有获取锁等。每当goroutine被这些操作阻塞时，剖析库都会记录相应的事件。

只需要开启下面其中一个标志参数就可以生成各种分析文件。当同时使用多个标志参数时需要当心，因为一项分析操作可能会影响其他项的分析结果。
```bash
$ go test -cpuprofile=cpu.out
$ go test -blockprofile=block.out
$ go test -memprofile=mem.out
```
对于一些非测试程序也很容易进行剖析，具体的实现方式，与程序是短时间运行的小工具还是长时间运行的服务会有很大不同。剖析对于长期运行的程序尤其有用，因此可以通过调用Go的runtime API来启用运行时剖析。

一旦已经收集到了用于分析的采样数据，就可以使用pprof来分析这些数据。这是Go工具箱自带的一个工具，但并不是一个日常工具，它对应`go tool pprof`命令。该命令有许多特性和选项，但是最基本的是两个参数：生成这个概要文件的可执行程序和对应的剖析数据。

为了提高分析效率和减少空间，分析日志本身并不包含函数的名字；它只包含函数对应的地址。也就是说pprof需要对应的可执行程序来解读剖析数据。虽然`go test`通常在测试完成后就丢弃临时用的测试程序，但是在启用分析的时候会将测试程序保存为foo.test文件，其中foo部分对应待测包的名字。

下面的命令演示了如何收集并展示一个CPU分析文件。我们选择`net/http`包的一个基准测试为例。通常最好是对业务关键代码的部分设计专门的基准测试。因为简单的基准测试几乎没法代表业务场景，因此我们用-run=NONE参数禁止那些简单测试。
```bash
$ go test -run=NONE -bench=ClientServerParallelTLS64 \
    -cpuprofile=cpu.log net/http
 PASS
 BenchmarkClientServerParallelTLS64-8  1000
    3141325 ns/op  143010 B/op  1747 allocs/op
ok       net/http       3.395s

$ go tool pprof -text -nodecount=10 ./http.test cpu.log
2570ms of 3590ms total (71.59%)
Dropped 129 nodes (cum <= 17.95ms)
Showing top 10 nodes out of 166 (cum >= 60ms)
    flat  flat%   sum%     cum   cum%
  1730ms 48.19% 48.19%  1750ms 48.75%  crypto/elliptic.p256ReduceDegree
   230ms  6.41% 54.60%   250ms  6.96%  crypto/elliptic.p256Diff
   120ms  3.34% 57.94%   120ms  3.34%  math/big.addMulVVW
   110ms  3.06% 61.00%   110ms  3.06%  syscall.Syscall
    90ms  2.51% 63.51%  1130ms 31.48%  crypto/elliptic.p256Square
    .....
```
参数-text用于指定输出格式，在这里每行是一个函数，根据使用CPU的时间长短来排序。其中-nodecount=10参数限制了只输出前10行的结果。对于严重的性能问题，这个文本格式基本可以帮助查明原因了。

这个概要文件告诉我们，HTTPS基准测试中crypto/elliptic.p256ReduceDegree函数占用了将近一半的CPU资源，对性能占很大比重。相比之下，如果一个概要文件中主要是runtime包的内存分配的函数，那么减少内存消耗可能是一个值得尝试的优化策略。

对于一些更微妙的问题，你可能需要使用pprof的图形显示功能。这个需要安装GraphViz工具，可以从 http://www.graphviz.org 下载。参数-web用于生成函数的有向图，标注有CPU的使用和最热点的函数等信息。

### 示例函数
示例函数，以Example为函数名开头。示例函数没有函数参数和返回值。下面是IsPalindrome函数对应的示例函数：
```go
func ExampleIsPalindrome() {
  fmt.Println(IsPalindrome("A man, a plan, a canal: Panama"))
  fmt.Println(IsPalindrome("palindrome"))
  // Output:
  // true
  // false
}
```
示例函数有三个用处。最主要的一个是作为文档：一个包的例子可以更简洁直观的方式来演示函数的用法，比文字描述更直接易懂，特别是作为一个提醒或快速参考时。一个示例函数也可以方便展示属于同一个接口的几种类型或函数之间的关系，所有的文档都必须关联到一个地方，就像一个类型或函数声明都统一到包一样。同时，示例函数和注释并不一样，示例函数是真实的Go代码，需要接受编译器的编译时检查，这样可以保证源代码更新时，示例代码不会脱节。

示例函数的第二个用处是，在`go test`执行测试的时候也会运行示例函数测试。如果示例函数内含有类似上面例子中的`// Output:`格式的注释，那么测试工具会执行这个示例函数，然后检查示例函数的标准输出与注释是否匹配。

示例函数的第三个目的提供一个真实的演练场。 http://golang.org 就是由godoc提供的文档服务，它使用了Go Playground让用户可以在浏览器中在线编辑和运行每个示例函数，

## 反射
Go语言提供了一种机制，能够在运行时更新变量和检查它们的值、调用它们的方法和它们支持的内在操作，而不需要在编译时就知道这些变量的具体类型。这种机制被称为反射。反射也可以让我们将类型本身作为第一类的值类型处理。

### reflect.Type 和 reflect.Value
反射是由 reflect 包提供的。reflect.Type 和 reflect.Value 是反射的两个核心类型。

#### reflect.Type
一个 Type 表示一个Go类型。它是一个接口，有许多方法来区分类型以及检查它们的组成部分，例如一个结构体的成员或一个函数的参数等。唯一能反映 reflect.Type 实现的是接口的类型描述信息，也正是这个实体标识了接口值的动态类型。

函数 reflect.TypeOf 接受任意的 interface{} 类型，并以 reflect.Type 形式返回其动态类型：
```go
t := reflect.TypeOf(3)  // a reflect.Type
fmt.Println(t.String()) // "int"
fmt.Println(t)          // "int"
```
其中 TypeOf(3) 调用将值 3 传给 interface{} 参数。将一个具体的值转为接口类型会有一个隐式的接口转换操作，它会创建一个包含两个信息的接口值：操作数的动态类型（这里是 int）和它的动态的值（这里是 3）。

因为 reflect.TypeOf 返回的是一个动态类型的接口值，它总是返回具体的类型。因此，下面的代码将打印 "*os.File" 而不是 "io.Writer"。
```go
var w io.Writer = os.Stdout
fmt.Println(reflect.TypeOf(w)) // "*os.File"
```
#### reflect.Value
一个 reflect.Value 可以装载任意类型的值。函数 reflect.ValueOf 接受任意的 interface{} 类型，并返回一个装载着其动态值的 reflect.Value。和 reflect.TypeOf 类似，reflect.ValueOf 返回的结果也是具体的类型，但是 reflect.Value 也可以持有一个接口值。
```go
v := reflect.ValueOf(3) // a reflect.Value
fmt.Println(v)          // "3"
fmt.Printf("%v\n", v)   // "3"
fmt.Println(v.String()) // NOTE: "<int Value>"
```

eflect.ValueOf 的逆操作是 reflect.Value.Interface 方法。它返回一个 interface{} 类型，装载着与 reflect.Value 相同的具体值：
```go
v := reflect.ValueOf(3) // a reflect.Value
x := v.Interface()      // an interface{}
i := x.(int)            // an int
fmt.Printf("%d\n", i)   // "3"
```
reflect.Value 和 interface{} 都能装载任意的值。所不同的是，一个空的接口隐藏了值内部的表示方式和所有方法，因此只有知道具体的动态类型才能使用类型断言来访问内部的值（就像上面那样），内部值无法访问。相比之下，一个 Value 则有很多方法来检查其内容，无论它的具体类型是什么。

reflect.Value 的 Kind 方法: kinds 类型却是有限的：Bool、String 和 所有数字类型的基础类型；Array 和 Struct 对应的聚合类型；Chan、Func、Ptr、Slice 和 Map 对应的引用类型；interface 类型；还有表示空值的 Invalid 类型。（空的 reflect.Value 的 kind 即为 Invalid。）

### 示例：Display，一个递归的值打印器
```go
func display(path string, v reflect.Value) {
	switch v.Kind() {
	case reflect.Invalid:
		fmt.Printf("%s = invalid\n", path)
	case reflect.Slice, reflect.Array:
		for i := 0; i < v.Len(); i++ {
			display(fmt.Sprintf("%s[%d]", path, i), v.Index(i))
		}
	case reflect.Struct:
		for i := 0; i < v.NumField(); i++ {
			fieldPath := fmt.Sprintf("%s.%s", path, v.Type().Field(i).Name)
			display(fieldPath, v.Field(i))
		}
	case reflect.Map:
		for _, key := range v.MapKeys() {
			display(fmt.Sprintf("%s[%s]", path,
				formatAtom(key)), v.MapIndex(key))
		}
	case reflect.Ptr:
		if v.IsNil() {
			fmt.Printf("%s = nil\n", path)
		} else {
			display(fmt.Sprintf("(*%s)", path), v.Elem())
		}
	case reflect.Interface:
		if v.IsNil() {
			fmt.Printf("%s = nil\n", path)
		} else {
			fmt.Printf("%s.type = %s\n", path, v.Elem().Type())
			display(path+".value", v.Elem())
		}
	default: // basic types, channels, funcs
		fmt.Printf("%s = %s\n", path, formatAtom(v))
	}
}
```
**Slice和数组**： 两种的处理逻辑是一样的。Len方法返回slice或数组值中的元素个数，Index(i)获得索引i对应的元素，返回的也是一个reflect.Value；如果索引i超出范围的话将导致panic异常，这与数组或slice类型内建的len(a)和a[i]操作类似。display针对序列中的每个元素递归调用自身处理，通过在递归处理时向path附加“[i]”来表示访问路径。

虽然reflect.Value类型带有很多方法，但是只有少数的方法能对任意值都安全调用。例如，Index方法只能对Slice、数组或字符串类型的值调用，如果对其它类型调用则会导致panic异常。

**结构体**： NumField方法报告结构体中成员的数量，Field(i)以reflect.Value类型返回第i个成员的值。成员列表也包括通过匿名字段提升上来的成员。为了在path添加“.f”来表示成员路径，必须获得结构体对应的reflect.Type类型信息，然后访问结构体第i个成员的名字。

**Maps**: MapKeys方法返回一个reflect.Value类型的slice，每一个元素对应map的一个key。和往常一样，遍历map时顺序是随机的。MapIndex(key)返回map中key对应的value。向path添加“[key]”来表示访问路径。

**指针**： Elem方法返回指针指向的变量，依然是reflect.Value类型。即使指针是nil，这个操作也是安全的，在这种情况下指针是Invalid类型，但是可以用IsNil方法来显式地测试一个空指针，这样可以打印更合适的信息。在path前面添加“*”，并用括弧包含以避免歧义。

**接口**： 使用IsNil方法来测试接口是否是nil，如果不是，可以调用v.Elem()来获取接口对应的动态值，并且打印对应的类型和值。

### 通过reflect.Value修改值
Go语言中类似x、x.f[1]和*p形式的表达式都可以表示变量，但是其它如x + 1和f(2)则不是变量。一个变量就是一个可寻址的内存空间，里面存储了一个值，并且存储的值可以通过内存地址来更新。

对于reflect.Values也有类似的区别。有一些reflect.Values是可取地址的；其它一些则不可以
```go
x := 2                   // value   type    variable?
a := reflect.ValueOf(2)  // 2       int     no
b := reflect.ValueOf(x)  // 2       int     no
c := reflect.ValueOf(&x) // &x      *int    no
d := c.Elem()            // 2       int     yes (x)
```
所有通过reflect.ValueOf(x)返回的reflect.Value都是不可取地址的。但是d是c的解引用方式生成的，指向另一个变量，因此是可取地址的。可以通过调用reflect.ValueOf(&x).Elem()，来获取任意变量x对应的可取地址的Value。

可以通过调用reflect.Value的CanAddr方法来判断其是否可以被取地址：
```go
fmt.Println(a.CanAddr()) // "false"
fmt.Println(b.CanAddr()) // "false"
fmt.Println(c.CanAddr()) // "false"
fmt.Println(d.CanAddr()) // "true"
```
通过指针间接地获取的reflect.Value都是可取地址的，即使开始的是一个不可取地址的Value。在反射机制中，所有关于是否支持取地址的规则都是类似的。例如，slice的索引表达式e[i]将隐式地包含一个指针，它就是可取地址的，即使开始的e表达式不支持也没有关系。以此类推，reflect.ValueOf(e).Index(i)对应的值也是可取地址的，即使原始的reflect.ValueOf(e)不支持也没有关系。

要从变量对应的可取地址的reflect.Value来访问变量需要三个步骤。第一步是调用Addr()方法，它返回一个Value，里面保存了指向变量的指针。然后是在Value上调用Interface()方法，也就是返回一个interface{}，里面包含指向变量的指针。最后，如果知道变量的类型，可以使用类型的断言机制将得到的interface{}类型的接口强制转为普通的类型指针。这样就可以通过这个普通指针来更新变量了：
```go
x := 2
d := reflect.ValueOf(&x).Elem()   // d refers to the variable x
px := d.Addr().Interface().(*int) // px := &x
*px = 3                           // x = 3
fmt.Println(x)                    // "3"
```
或者，不使用指针，而是通过调用可取地址的reflect.Value的reflect.Value.Set方法来更新对应的值：
```go
d.Set(reflect.ValueOf(4))
fmt.Println(x) // "4"
```
Set方法将在运行时执行和编译时进行类似的可赋值性约束的检查。基本数据类型的Set方法：SetInt、SetUint、SetString和SetFloat等。
```go
// 变量和值都是int类型，但是如果变量是int64类型，那么程序将抛出一个panic异常，
d.Set(reflect.ValueOf(int64(5))) // panic: int64 is not assignable to int

// 对一个不可取地址的reflect.Value调用Set方法也会导致panic异常
x := 2
b := reflect.ValueOf(x)
b.Set(reflect.ValueOf(3)) // panic: Set using unaddressable value
```

一个可取地址的reflect.Value会记录一个结构体成员是否是未导出成员，如果是的话则拒绝修改操作。因此，CanAddr方法并不能正确反映一个变量是否是可以被修改的。另一个相关的方法CanSet是用于检查对应的reflect.Value是否是可取地址并可被修改的：
```go
fmt.Println(fd.CanAddr(), fd.CanSet()) // "true false"
```

### 示例：获取结构体字段标签
Unpack函数将构建每个结构体成员有效参数名字到成员变量的映射。如果结构体成员有成员标签的话，有效参数名字可能和实际的成员名字不相同。reflect.Type的Field方法将返回一个reflect.StructField，里面含有每个成员的名字、类型和可选的成员标签等信息。其中成员标签信息对应reflect.StructTag类型的字符串，并且提供了Get方法用于解析和根据特定key提取的子串。
```go
// Unpack 从 req 中的 HTTP 请求参数填充 ptr 指向的结构体的字段。
func Unpack(req *http.Request, ptr interface{}) error {
	if err := req.ParseForm(); err != nil {
		return err
	}

	// 构建以有效名称为键的map。
	fields := make(map[string]reflect.Value)
	v := reflect.ValueOf(ptr).Elem() // 结构变量
	for i := 0; i < v.NumField(); i++ {
		fieldInfo := v.Type().Field(i) // 一个reflect.StructField
		tag := fieldInfo.Tag           // 一个 reflect.StructTag
		name := tag.Get("http")
		if name == "" {
			name = strings.ToLower(fieldInfo.Name)
		}
		fields[name] = v.Field(i)
	}

	// 更新请求中每个参数的结构字段。
	for name, values := range req.Form {
		f := fields[name]
		if !f.IsValid() {
			continue // 忽略无法识别的 HTTP 参数
		}
		for _, value := range values {
			if f.Kind() == reflect.Slice {
				elem := reflect.New(f.Type().Elem()).Elem()
				if err := populate(elem, value); err != nil {
					return fmt.Errorf("%s: %v", name, err)
				}
				f.Set(reflect.Append(f, elem))
			} else {
				if err := populate(f, value); err != nil {
					return fmt.Errorf("%s: %v", name, err)
				}
			}
		}
	}
	return nil
}
```
最后，Unpack遍历HTTP请求的name/valu参数键值对，并且根据更新相应的结构体成员。回想一下，同一个名字的参数可能出现多次。如果发生这种情况，并且对应的结构体成员是一个slice，那么就将所有的参数添加到slice中。其它情况，对应的成员值将被覆盖，只有最后一次出现的参数值才是起作用的。

### 示例：显示一个类型的方法集
```go
// Print 打印值 x 的方法集。
func Print(x interface{}) {
	v := reflect.ValueOf(x)
	t := v.Type()
	fmt.Printf("type %s\n", t)

	for i := 0; i < v.NumMethod(); i++ {
		methType := v.Method(i).Type()
		fmt.Printf("func (%s) %s%s\n", t, t.Method(i).Name,
			strings.TrimPrefix(methType.String(), "func"))
	}
}
```
reflect.Type和reflect.Value都提供了一个Method方法。每次t.Method(i)调用将一个reflect.Method的实例，对应一个用于描述一个方法的名称和类型的结构体。每次v.Method(i)方法调用都返回一个reflect.Value以表示对应的值，也就是一个方法是帮到它的接收者的。使用reflect.Value.Call方法，将可以调用一个Func类型的Value，但是这个例子中只用到了它的类型。

> 反射是一个强大并富有表达力的工具，但是它应该被小心地使用，原因有三。
> 1. 基于反射的代码是比较脆弱的。对于每一个会导致编译器报告类型错误的问题，在反射中都有与之相对应的误用问题，不同的是编译器会在构建时马上报告错误，而反射则是在真正运行到的时候才会抛出panic异常，可能是写完代码很久之后了，而且程序也可能运行了很长的时间。
> 2. 即使对应类型提供了相同文档，但是反射的操作不能做静态类型检查，而且大量反射的代码通常难以理解。总是需要小心翼翼地为每个导出的类型和其它接受interface{}或reflect.Value类型参数的函数维护说明文档。
> 3. 基于反射的代码通常比正常的代码运行速度慢一到两个数量级。

## 底层编程

### unsafe.Sizeof, Alignof 和 Offsetof
unsafe.Sizeof函数返回操作数在内存中的字节大小，参数可以是任意类型的表达式，但是它并不会对表达式进行求值。一个Sizeof函数调用是一个对应uintptr类型的常量表达式，因此返回的结果可以用作数组类型的长度大小，或者用作计算其他的常量。
```go
import "unsafe"
fmt.Println(unsafe.Sizeof(float64(0))) // "8"
```
Sizeof函数返回的大小只包括数据结构中固定的部分，例如字符串对应结构体中的指针和字符串长度部分，但是并不包含指针指向的字符串的内容。Go语言中非聚合类型通常有一个固定的大小，尽管在不同工具链下生成的实际大小可能会有所不同。考虑到可移植性，引用类型或包含引用类型的大小在32位平台上是4个字节，在64位平台上是8个字节。

计算机在加载和保存数据时，如果内存地址合理地对齐的将会更有效率。例如2字节大小的int16类型的变量地址应该是偶数，一个4字节大小的rune类型变量的地址应该是4的倍数，一个8字节大小的float64、uint64或64-bit指针类型变量的地址应该是8字节对齐的。但是对于再大的地址对齐倍数则是不需要的，即使是complex128等较大的数据类型最多也只是8字节对齐。

由于地址对齐这个因素，一个聚合类型（结构体或数组）的大小至少是所有字段或元素大小的总和，或者更大因为可能存在内存空洞。内存空洞是编译器自动添加的没有被使用的内存空间，用于保证后面每个字段或元素的地址相对于结构或数组的开始地址能够合理地对齐（内存空洞可能会存在一些随机数据，可能会对用unsafe包直接操作内存的处理产生影响）。
|              类型             |                大小               |
|:-----------------------------:|:---------------------------------:|
| bool                          | 1个字节                           |
| intN, uintN, floatN, complexN | N/8个字节（例如float64是8个字节） |
| int, uint, uintptr            | 1个机器字                         |
| *T                            | 1个机器字                         |
| string                        | 2个机器字（data、len）            |
| []T                           | 3个机器字（data、len、cap）       |
| map                           | 1个机器字                         |
| func                          | 1个机器字                         |
| chan                          | 1个机器字                         |
| interface                     | 2个机器字（type、value）          |

`unsafe.Alignof` 函数返回对应参数的类型需要对齐的倍数。和 Sizeof 类似， Alignof 也是返回一个常量表达式，对应一个常量。通常情况下布尔和数字类型需要对齐到它们本身的大小（最多8个字节），其它的类型对齐到机器字大小。

`unsafe.Offsetof` 函数的参数必须是一个字段 `x.f`，然后返回 `f` 字段相对于 `x` 起始地址的偏移量，包括可能的空洞。
```go
var x struct {
  a bool
  b int16
  c []int
}
```
下面显示了对x和它的三个字段调用unsafe包相关函数的计算结果: 
![计算结果](img-01.png)
<!-- {% asset_img img-01.png 计算结果 %} -->
```go
// 32位系统：
Sizeof(x)   = 16  Alignof(x)   = 4
Sizeof(x.a) = 1   Alignof(x.a) = 1 Offsetof(x.a) = 0
Sizeof(x.b) = 2   Alignof(x.b) = 2 Offsetof(x.b) = 2
Sizeof(x.c) = 12  Alignof(x.c) = 4 Offsetof(x.c) = 4

// 64位系统：
Sizeof(x)   = 32  Alignof(x)   = 8
Sizeof(x.a) = 1   Alignof(x.a) = 1 Offsetof(x.a) = 0
Sizeof(x.b) = 2   Alignof(x.b) = 2 Offsetof(x.b) = 2
Sizeof(x.c) = 24  Alignof(x.c) = 8 Offsetof(x.c) = 8
```

### unsafe.Pointer
大多数指针类型会写成*T，表示是“一个指向T类型变量的指针”。unsafe.Pointer是特别定义的一种指针类型，它可以包含任意类型变量的地址。当然，我们不可以直接通过*p来获取unsafe.Pointer指针指向的真实变量的值，因为我们并不知道变量的具体类型。和普通指针一样，unsafe.Pointer指针也是可以比较的，并且支持和nil常量比较判断是否为空指针。

一个普通的*T类型指针可以被转化为unsafe.Pointer类型指针，并且一个unsafe.Pointer类型指针也可以被转回普通的指针，被转回普通的指针类型并不需要和原始的*T类型相同。通过将*float64类型指针转化为*uint64类型指针，可以查看一个浮点数变量的位模式。
```go
package math

func Float64bits(f float64) uint64 { return *(*uint64)(unsafe.Pointer(&f)) }

fmt.Printf("%#016x\n", Float64bits(1.0)) // "0x3ff0000000000000"
```
通过转为新类型指针，可以更新浮点数的位模式。通过位模式操作浮点数是可以的，但是更重要的意义是指针转换语法让我们可以在不破坏类型系统的前提下向内存写入任意的值。

一个unsafe.Pointer指针也可以被转化为uintptr类型，然后保存到指针型数值变量中（这只是和当前指针相同的一个数字值，并不是一个指针），然后用以做必要的指针数值运算。（uintptr是一个无符号的整型数，足以保存一个地址）这种转换虽然也是可逆的，但是将uintptr转为unsafe.Pointer指针可能会破坏类型系统，因为并不是所有的数字都是有效的内存地址。

许多将unsafe.Pointer指针转为原生数字，然后再转回为unsafe.Pointer类型指针的操作也是不安全的。比如下面的例子需要将变量x的地址加上b字段地址偏移量转化为*int16类型指针，然后通过该指针更新x.b：
```go
gopl.io/ch13/unsafeptr

var x struct {
  a bool
  b int16
  c []int
}

// 和 pb := &x.b 等价
pb := (*int16)(unsafe.Pointer(
    uintptr(unsafe.Pointer(&x)) + unsafe.Offsetof(x.b)))
*pb = 42
fmt.Println(x.b) // "42"
```
上面的写法尽管很繁琐，但在这里并不是一件坏事，因为这些功能应该很谨慎地使用。不要试图引入一个uintptr类型的临时变量，因为它可能会破坏代码的安全性（这是真正可以体会unsafe包为何不安全的例子）。下面段代码是错误的：
```go
// NOTE: subtly incorrect!
tmp := uintptr(unsafe.Pointer(&x)) + unsafe.Offsetof(x.b)
pb := (*int16)(unsafe.Pointer(tmp))
*pb = 42
```
产生错误的原因很微妙。有时候垃圾回收器会移动一些变量以降低内存碎片等问题。这类垃圾回收器被称为移动GC。当一个变量被移动，所有的保存该变量旧地址的指针必须同时被更新为变量移动后的新地址。从垃圾收集器的视角来看，一个unsafe.Pointer是一个指向变量的指针，因此当变量被移动时对应的指针也必须被更新；但是uintptr类型的临时变量只是一个普通的数字，所以其值不应该被改变。上面错误的代码因为引入一个非指针的临时变量tmp，导致垃圾收集器无法正确识别这个是一个指向变量x的指针。当第二个语句执行时，变量x可能已经被转移，这时候临时变量tmp也就不再是现在的&x.b地址。第三个向之前无效地址空间的赋值语句将彻底摧毁整个程序！

还有很多类似原因导致的错误。例如这条语句：
```go
pT := uintptr(unsafe.Pointer(new(T))) // 提示: 错误!
```
这里并没有指针引用new新创建的变量，因此该语句执行完成之后，垃圾收集器有权马上回收其内存空间，所以返回的pT将是无效的地址。

### 通过cgo调用C代码
```go
// bzip 包提供了一个使用 bzip2 压缩（bzip.org）的编写器。
package bzip

/*
#cgo CFLAGS: -I/usr/include
#cgo LDFLAGS: -L/usr/lib -lbz2
#include <bzlib.h>
#include <stdlib.h>
bz_stream* bz2alloc() { return calloc(1, sizeof(bz_stream)); }
int bz2compress(bz_stream *s, int action,
                char *in, unsigned *inlen, char *out, unsigned *outlen);
void bz2free(bz_stream* s) { free(s); }
*/
import "C"  // import "C"的语句是比较特别的。其实并没有一个叫C的包，但是这行语句会让Go编译程序在编译之前先运行cgo工具。

import (
  "io"
  "unsafe"
)

type writer struct {
  w      io.Writer // 底层输出流
  stream *C.bz_stream
  outbuf [64 * 1024]byte
}

// NewWriter 返回一个用于 bzip2 压缩流的写入器。
func NewWriter(out io.Writer) io.WriteCloser {
  const blockSize = 9
  const verbosity = 0
  const workFactor = 30
  w := &writer{w: out, stream: C.bz2alloc()}
  C.BZ2_bzCompressInit(w.stream, blockSize, verbosity, workFactor)
  return w
}
```
在预处理过程中，cgo工具生成一个临时包用于包含所有在Go语言中访问的C语言的函数或类型。在cgo注释中还可以包含#cgo指令，用于给C语言工具链指定特殊的参数。例如CFLAGS和LDFLAGS分别对应传给C语言编译器的编译参数和链接器参数，使它们可以从特定目录找到bzlib.h头文件和libbz2.a库文件。
