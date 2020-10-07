### 文件名、关键字与标识符
#### 文件名
`Go`的源文件以`.go`结尾，如`scanner.go`,如果存在多个部分组成的话，用下划线`_`对其进行分割，如`scanner_test.go`

#### 关键字
Go只有`25`个关键字，简化设计
<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>break</td>
    <td>default</td>
    <td>func</td>
    <td>interface</td>
    <td>select</td>
  </tr>
  <tr>
    <td>case</td>
    <td>defer</td>
    <td>go</td>
    <td>map</td>
    <td>struct</td>
  </tr>
  <tr>
    <td>chan</td>
    <td>else</td>
    <td>goto</td>
    <td>package</td>
    <td>switch</td>
  </tr>
  <tr>
    <td>const</td>
    <td>fallthrough</td>
    <td>if</td>
    <td>range</td>
    <td>type</td>
  </tr>
  <tr>
    <td>continue</td>
    <td>for</td>
    <td>import</td>
    <td>return</td>
    <td>var</td>
  </tr>
</table>

#### 标识符
**无效标识符**
1. 1ab(以数字开头)
2. case（Go 语言的关键字）
3. a+b（运算符是不允许的）

程序一般由关键字、常量、变量、运算符、类型和函数组成。

程序中可能会使用到这些分隔符：括号`()`，中括号`[]`和大括号`{}`

程序中可能会使用到这些标点符号：`.、,、;、: 和 …`


### 1.第一个程序
```
package main

import "fmt"

func main() {
	fmt.Println("hello, world")
}
```
#### 1.1 go程序结构

go程序由`package` `import` 和`funcion`这三部分构成

`package main`表示一个可独立执行的程序，每个Go应用程序都包含一个名为 `main`的包 这是程序入口的地方

一个程序可以包含不同的包，在每个文件上面指明`package`名就行

`如果包名不是以 . 或 / 开头，如 "fmt" 或者 "container/list"，则 Go 会在全局文件进行查找；如果包名以 ./ 开头，则 Go 会在相对目录中查找；如果包名以 / 开头（在 Windows 下也可以这样使用），则会在系统的绝对路径中查找`

#### 1.2 可见性规则

首字母大写**可以被外部包**使用，首字母小写**只能整个包**使用

当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 private ）

#### 1.3 函数
函数的定义
```
func functionName()
func functionName(parameter_list) (return_value_list) {
   …
}

```
**特殊函数**
`main`函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数（如果有`init()` 函数则会先执行该函数）
执行顺序 `init()`先于`main()`

#### 1.4注释
单行注释
```
// helpful when attempting to save the world.
```
多行注释
```
/* 
helpful when attempting to save the world.
helpful when attempting to save the world.
*/
```
#### 1.5 go程序启动顺序
1. 按顺序导入所有被`main` 包引用的其它包，然后在每个包中执行如下流程
2. 如果该包又导入了其它的包，则从第一步开始递归执行，但是每个包只会被导入一次
3. 然后以相反的顺序在每个包中初始化常量和变量，如果该包含有 `init`函数的话，则调用该函数 
4. 在完成这一切之后，`main` 也执行同样的过程，最后调用`main` 函数开始执行程序

#### 1.6 类型转换
转换格式
```
valueOfTypeB = typeB(valueOfTypeA)
```
将`float` 转换成`int`
```
a := 5.0
b := int(a) 
```

#### 常量
常量使用`const`关键字定义

数据类型只能是：`布尔型`、`数字型（整数型、浮点型和复数）`和`字符串型`

常量赋值实例：
```
const (
	Unknown = 0
	Female = 1
	Male = 2
)
```
注意：常量的值必须是能够在编译时就能够确定的，错误的做法：

```
const c2 = getNumber() 
```
引发构建错误: `getNumber() used as value`**因为在编译期间自定义函数均属于未知，因此无法用于常量的赋值，但内置函数可以使用，如：len()**



#### 变量
声明变量的一般形式为`var` 关键字
`var identifier type`

**变量类型**

基本类型，如：`int、float、bool、string`；结构化的（复合的），如：`struct、array、slice、map、channel`；只描述类型的行为的，如：`interface`

结构化的类型没有真正的值，它使用 nil 作为**默认值**

类型往往可以省略，因为**Go 编译器的智商已经高到可以根据变量的值来自动推断其类型** 这与python
是相似的
实例：
```
var a int = 15  等价于  var a = 15
```


**全局变量声明**
```
var a = 15
var b = false
var str = "Go says hello to the world!"
```
**推荐**更简洁的定义方式：因式分解关键字  
```
var (
    a = 15
    b = false
    str = "Go says hello to the world!"
)
```
**局部变量声明**

在函数体内声明变量，应该简短使用
`:=`s 省略`var`  例如：
```
a := 1  等价于 var a=1
```

#### 值类型和引用类型
- **值类型**：变量里面存储的是值 **基本类型**都属于值类型 还有**数组和结构体**
- **引用类型**：变量名存储的地址，**复合类型**都属于引用类型

#### 运算符
非运算符：`!`
```
!T -> false
!F -> true
```
和运算符：`&&`
```
T && T -> true
T && F -> false
F && T -> false
F && F -> false
```
或运算符：`||`
```
T || T -> true
T || F -> true
F || T -> true
F || F -> false
```
在 Go 语言中，&& 和 || 是具有**快捷性质的运算符**，当运算符左边表达式的值已经能够决定整个表达式的值的时候（&& 左边的值为 false，|| 左边的值为 true），运算符右边的表达式将不会被执行)

#### 字符串

字符串是一种**值类型，且值不可变**，即创建某个文本后你无法再次修改这个文本的内容，更深入地讲，字符串是**字节的定长数组**
1.  **解释字符串**
该类字符串使用**双引号**括起来，其中的相关的转义字符将被替换，这些转义字符包括：
    - `\n`：换行符
    - `\r`：回车符
    - `\t`：tab 键
    - `\u` 或 \U：Unicode 字符
    - `\\`：反斜杠自身
2. 非解释字符串
该类字符串使用**反引号**括起来，支持换行，例如
```
  `This is a raw string \n` 中的 `\n\` 会被原样输出。
 
```
空字符串 `""`
- 字符串 str 的第 1 个字节：`str[0]`
- 第 i 个字节：`str[i - 1]`
- 最后 1 个字节：`str[len(str)-1]`

字符串拼接 `+`
```
str := "Beginning of the string " +
	"second part of the string"
```

####  strings包
该包是`go`语言提供对字符串操作的主要方法
- `HasPrefix` 判断字符串是否以某某前缀开头
- `Contains` 字符串是否有包含关系
- `TrimSpace()` 来剔除字符串开头和结尾的空白符号,`strings.Trim(s, "cut")` 来将开头和结尾的 `cut`去除掉
- `strings.Fields(s)` 以空白符分割字符串
- `strings.Split(s, sep)` 自定义 分割符分割字符串
- `join` 用于将元素为 string 的 slice 使用分割符号来拼接组成一个字符串

####  指针
指针：就是变量里面存储的是**内存地址**，而不是值,go语言提供指针的支持，这与`java`不一样
几个知识点
- 声明一个指针变量 `var intP *int`必须有*
- 取内存地址的符号`&` 
- 取指针变量里面对应的真实值：在指针变量前面加`*` 比如`*intP`
- 当一个指针被定义后没有分配到任何变量时，它的值为 `nil`
```
package main
import "fmt"
func main() {
	var i1 = 5
	fmt.Printf("An integer: %d, its location in memory: %p\n", i1, &i1)
	var intP *int 
	intP = &i1  //取变量内存值赋给指针
	fmt.Printf("The value at memory location %p is %d\n", intP, *intP)  //取指针指向的内容
}
```
output:
```
An integer: 5, its location in memory: 0x24f0820
The value at memory location 0x24f0820 is 5
```
**注意事项：**
不能获取字面量或常量的地址
```
const i = 5
ptr := &i //error: cannot take the address of i
ptr2 := &10 //error: cannot take the address of 10
```

### 控制结构
#### if else
```
if condition1 {
	// do something	
} else if condition2 {
	// do something else	
} else {
	// catch-all or default
}
```
布尔值类型判断 不需要加`bool1==true`
```
package main
import "fmt"
func main() {
	bool1 := true
	if bool1 {  // 不需要加`bool1==true`
		fmt.Printf("The value is true\n")
	} else {
		fmt.Printf("The value is false\n")
	}
}
```
判断字符串是否为空
```
if str == "" { ... }
if len(str) == 0 {...}
```
if 可以包含一个`初始化语句`
```
if initialization; condition {
	// do something
}
```
例如：
```
val := 10
if val > max {
	// do something
}
```
可以写成这样
```
if val := 10; val > max {
	// do something
}
```

#### 测试多返回值函数的错误

Go 语言的函数经常使用**两个返回值来表示执行是否成功**：返回某个值以及 true 表示成功；返回零值（或 nil）和 false 表示失败

当不使用 true 或 false 的时候，也可以使用一个`error` 类型的变量来代替作为第二个返回值：**成功执行的话，error 的值为 nil，否则就会包含相应的错误信息** 

`_` 忽略对error的检查

也就是我们的代码要对`error`进行校验
```
value, err := pack1.Function1(param1)
if err != nil {
	fmt.Printf("An error occured in pack1.Function1 with parameter %v", param1)
	return err
	或者
	os.Exit(1)  //终止程序的运行
}
// 未发生错误，继续执行：
```
```
if err := file.Chmod(0664); err != nil {
	fmt.Println(err)
	return err
}
```

```
func mySqrt(f float64) (v float64, ok bool) {
	if f < 0 { return } // error case
	return math.Sqrt(f),true
}

func main() {
	res, ok := mySqrt(25.0)
	if !ok{
	    fmt.Println("abnormal")
	}
}
``` 
`_` 忽略对error的检查 相信一定不会出错
```
func atoi (s string) (n int) {
	n, _ = strconv.Atoi(s)
	return
}
```

#### switch语句
感受下大体的样子
```
package main

import "fmt"

func main() {
	var num1 int = 7

	switch {
	    case num1 < 0:
		    fmt.Println("Number is negative")
	    case num1 > 0 && num1 < 10:
		    fmt.Println("Number is between 0 and 10")
	    default:
		    fmt.Println("Number is 10 or greater")
	}
}
```
#### for循环 
go语言中没有`while`循环，而for循环可以做**一切重复的事情**
1. **基于计数器的迭代 基本格式**
```
for 初始化语句; 条件语句; 修饰语句 {}
```
```
package main

import "fmt"

func main() {
	for i := 0; i < 5; i++ {
		fmt.Printf("This is the %d iteration\n", i)
	}
}
```
2. **基于条件判断的迭代**
类似`while`循环
```
for 条件语句 {}
```
```
package main

import "fmt"

func main() {
	var i int = 5

	for i >= 0 {
		i = i - 1
		fmt.Printf("The variable i is now: %d\n", i)
	}
}
```
3. **无限循环**
条件语句是可以被省略的,如果`for` 循环的头部没有条件语句，那么就会认为条件永远为`true` 格式为
```
for { }
```
4. **for-range 结构**
该结构用于迭代集合类对象
```
for ix, val := range coll { }
```

#### 标签与 goto
标签与`goto`一起来实现循环
```
package main

import "fmt"

func main() {

LABEL1:
	for i := 0; i <= 5; i++ {
		for j := 0; j <= 5; j++ {
			if j == 4 {
				continue LABEL1
			}
			fmt.Printf("i is: %d, and j is: %d\n", i, j)
		}
	}

}
```
### 函数

Go是**编译型语言**，所以函数编写的顺序是无关紧要的；鉴于可读性的需求，最好把`main()` 函数写在文件的前面，其他函数按照一定逻辑顺序进行编写（例如函数被调用的顺序)

Go 里面有三种类型的函数：

- 普通的带有名字的函数
- 匿名函数或者lambda函数
- 方法

函数也可以以申明的方式被使用，作为**一个函数类型**
```
type binOp func(int, int) int
```
在这里，不需要函数体 `{}`

函数是一等值（first-class value）：它们可以赋值给变量，就像`add := binOp` 一样

#### 函数参数与返回值

多值返回是`go`语言的一大特性，相比于其他语言，可用于判断程序运行是否正确

**按值传递（call by value） 按引用传递（call by reference）**

值类型的传参：
`Go`默认使用按**值传递**来传递参数，也就是传递参数的副本,函数接收参数副本之后，在使用变量的过程中可能对副本的值进行更改，但不会影响到原来的变量。

如果希望可以直接修改参数的值，则可以传递内存地址(也就是指针)，变量名前面添加&符号，比如 `&variable`

引用类型：

在函数调用时，像`切片（slice）`、`字典`（map）、`接口`（interface）、`通道（channel）`这样的引用类型都是默认使用引用传递（即使没有显式的指出指针。

**命名的返回值**
```
package main

import "fmt"

var num int = 10
var numx2, numx3 int

func main() {
    numx2, numx3 = getX2AndX3(num)
    PrintValues()
    numx2, numx3 = getX2AndX3_2(num)
    PrintValues()
}

func PrintValues() {
    fmt.Printf("num = %d, 2x num = %d, 3x num = %d\n", num, numx2, numx3)
}

func getX2AndX3(input int) (int, int) {
    return 2 * input, 3 * input
}

func getX2AndX3_2(input int) (x2 int, x3 int) {
    x2 = 2 * input
    x3 = 3 * input
    // return x2, x3
    return
}
```
`return` 后面没有跟具体的`变量`,变量在函数声明里

**空白符_**
作用：匹配一些不需要的值，然后丢弃掉，`_`代表不关注这个值
```
package main

import "fmt"

func main() {
    var i1 int
    var f1 float32
    i1, _, f1 = ThreeValues()
    fmt.Printf("The int: %d, the float: %f \n", i1, f1)
}

func ThreeValues() (int, int, float32) {
    return 5, 6, 7.5
}

res:
The int: 5, the float: 7.500000
```

**改变外部变量**
通过指针的方式改变外部变量
```
package main

import (
    "fmt"
)

// this function changes reply:
func Multiply(a, b int, reply *int) {
    *reply = a * b
}

func main() {
    n := 0
    reply := &n
    Multiply(10, 5, reply)
    fmt.Println("Multiply:", *reply) // Multiply: 50
}
```
#### 传递变长参数
如果函数的**最后一个参数**是采用 `...type`的形式，那么这个函数就可以处理一个变长的参数，这个长度可以为`0`，这样的函数称为变参函数
```
func Greeting(prefix string, who ...string)
Greeting("hello:", "Joe", "Anna", "Eileen")
```
```
在 Greeting 函数中，变量 who 的值为 []string{"Joe", "Anna", "Eileen"}
```
#### defer 和跟踪
关键字`defer`允许我们推迟到函数返回之前（或任意位置执行 return 语句之后）一刻才执行某个语句或函数（为什么要在返回之后才执行这些语句？因为 return 语句同样可以包含一些操作，而不是单纯地返回某个值） 

即调用`defer`的当前函数最后执行的一条语句

主要有两个用途：
1. 用来执行一些收尾工作，如关闭连接
2. 用`defer`来实现代码追踪
```
package main

import "fmt"

func main() {
	doDBOperations()
}

func connectToDB() {
	fmt.Println("ok, connected to db")
}

func disconnectFromDB() {
	fmt.Println("ok, disconnected from db")
}

func doDBOperations() {
	connectToDB()
	fmt.Println("Defering the database disconnect.")
	defer disconnectFromDB() //function called here with defer
	fmt.Println("Doing some DB operations ...")
	fmt.Println("Oops! some crash or network error ...")
	fmt.Println("Returning from function here!")
	return //terminate the program
	// deferred function executed here just before actually returning, even if
	// there is a return or abnormal termination before
}
```
代码跟踪
```
package main

import "fmt"

func trace(s string) string {
	fmt.Println("entering:", s)
	return s
}

func un(s string) {
	fmt.Println("leaving:", s)
}

func a() {
	defer un(trace("a"))
	fmt.Println("in a")
}

func b() {
	defer un(trace("b"))
	fmt.Println("in b")
	a()
}

func main() {
	b()
}
```

#### 内置函数
new、make

**相同点：**

用于分配内存，
用法都像函数，但将类型作为参数new(type)、make(type) 

**不同点：**
- new 用于**值类型和用户定义的类型** new(T) 分配类型 T 的**零值并返回其地址**，也就是指向类型 T 的指针
- make  用于**内置引用类型**（切片、map 和管道） **make(T) 返回类型T的初始化之后的值**，因此它比 new 进行更多的工作

#### 将函数作为参数
函数可以作为其它函数的参数进行传递，然后在其它函数内调用执行，一般称之为回调
```
package main

import (
	"fmt"
)

func main() {
	callback(1, Add)
}

func Add(a, b int) {
	fmt.Printf("The sum of %d and %d is: %d\n", a, b, a+b)
}

func callback(y int, f func(int, int)) {
	f(y, 2) // this becomes Add(1, 2)
}
```

#### 匿名函数
只是没有名字
```
func() {
	sum := 0
	for i := 1; i <= 1e6; i++ {
		sum += i
	}
}()
```
表示参数列表的第一对括号**必须紧挨着关键字 func**，因为匿名函数没有名称。花括号 {} 涵盖着函数体，最后的一对括号`()`表示对该匿名函数的调用

匿名函数的调用，有两种方式
1. 在匿名函数定义后通过`()`
2. 匿名函数可以被赋值于某个变量，即保存函数的地址到变量中，然后通过变量名对函数进行调用：`fplus(3,4)`
```
package main

import "fmt"

func main() {
	f()
}
func f() {
	for i := 0; i < 4; i++ {
		g := func(i int) { fmt.Printf("%d ", i) } //此例子中只是为了演示匿名函数可分配不同的内存地址，在现实开发中，不应该把该部分信息放置到循环中。
		g(i)
		fmt.Printf(" - g is of type %T and has value %v\n", g, g)
	}
}
```
匿名函数和`defer`一起用
```
package main

import "fmt"

func f() (ret int) {
	defer func() {
		ret++
	}()
	return 1
}
func main() {
	fmt.Println(f())
}

result is 2
```

### 数组和slice
#### 数组
数组是具有相同**唯一类型** 的一组已编号且**长度固定**的数据项序列（这是一种同构的数据结构）


Go 语言中的数组是一种 **值类型**（不像 C/C++ 中是指向首元素的指针），所以可以通过`new()`来创建: `var arr1 = new([5]int)`

数组的定义 必须指明长度
```
var identifier [len]type
```
举例：
```
var arr1 [5]int
```
由于数组的特性，所以在go语言中并不使用，而经常使用的是`slice`(切片)

#### slice
`slice`是`引用类型` 类似python中的`list`，是**长度可变**

`cap()` 获取slice的容量 元素最多的可以达到大小
`len()` 获取当前slice的大小

定义格式：
```
var identifier []type（不需要说明长度）
```
初始化格式
```
var slice1 []type = arr1[start:end]
```

但更常用的格式是通过`make()`来创建一个分片

`make`的使用方式是:`unc make([]T, len, cap)`，其中 cap 是可选参数。
```
定义一：
s2 := make([]int, 10) 
其中
cap(s2) == len(s2) == 10
定义二：
s3 :=make([]int, 50, 100)

cap(s3) == 100
len(s3) == 50
```
使用`new()`也可以创建切片，但尽量不要这么做

创建的是`数组`还是`slice`，最主要的是否**指定长度**

make只适用`切片、map 和 channel` 这三种类型

1. **slice、map以及channel**都是golang内建的一种引用类型，三者在内存中存在多个组成部分， 需要对内存组成部分初始化后才能使用，而make就是对三者进行初始化的一种操作方式
2. new 获取的是存储指定变量内存地址的一个变量，对于变量内部结构并不会执行相应的初始化操作， 所以slice、map、channel需要make进行初始化并获取对应的内存地址，而非new简单的获取内存地址

#### For-range 结构
```
for ix, value := range slice1 {
	...
}
```
第一个返回值 `ix` 是数组或者切片的索引，第二个是在该**索引位置的值**
```
package main
import "fmt"

func main() {
	seasons := []string{"Spring", "Summer", "Autumn", "Winter"}
	for ix, season := range seasons {
		fmt.Printf("Season %d is: %s\n", ix, season)
	}

	var season string
	for _, season = range seasons {
		fmt.Printf("%s\n", season)
	}
}
```
可以通过`_`忽略索引 比如注意这里没有`:`
```
for _, season = range seasons 
```
**只要索引**
```
for ix := range seasons {
	fmt.Printf("%d", ix)
}
```

#### 切片重组
```
var ar = [10]int{0,1,2,3,4,5,6,7,8,9}
var a = ar[5:7] // reference to subarray {5,6} - len(a) is 2 and cap(a) is 5
```
```
a = a[0:4] // ref of subarray {5,6,7,8} - len(a) is now 4 but cap(a) is still 5
```
这里前后两个a得出来的结果是不一样的，非常有意思。

#### slice 追加
```
	sl3 := []int{1, 2, 3}
	sl3 = append(sl3, 4, 5, 6)
	fmt.Println(sl3)
```
`func append(s[]T, x ...T)` 第一个参数是被追加的`slice`,第二个是追加的内容


slice切片的应用
https://github.com/unknwon/the-way-to-go_ZH_CN/blob/master/eBook/07.6.md

#### map
`map`是键值对组成的结构，类似python中的`dict`

`map`是引用类型,
`key`可以是任意可以用 == 或者 != 操作符比较的类型，比如 `string`、`int`、`float`,但不能使用`数组、切片和结构体不能作为 key`

但值`value`却可以是任意类型的

**性能：**

通过 key 在 map 中寻找值是很快的，比线性查找快得多，但是仍然比从数组和切片的索引中直接读取要`慢100`倍

`map`的长度是可以伸缩的

map定义
```
map1 := make(map[keytype]valuetype)
```
初始化
```
mapLit = map[string]int{"one": 1, "two": 2}
```
通过
`map1[key1] = val1` 访问或者设置值

#### map容量
map的容量可以根据新增的`key-value` 对动态的伸缩，但依然可以选择标明 map 的初始容量 capacity，就像这样：`make(map[keytype]valuetype, cap)`。例如：
```
map2 := make(map[string]float32, 100)
```
当 map 增长到容量上限的时候，如果再增加新的`key-value`对,map 的大小会自动加 1

#### 测试键值对是否存在及删除元素

使用`val1 = map1[key1]` 的方法获取 `key1` 对应的值 `val1`。如果 `map` 中不存在 `key1`，val1就是一个值类型的空值

判断key值是否存在？

```
val1, isPresent = map1[key1] 第二个字段是告诉你是否存在的
```
```
_, ok := map1[key1] // 如果key1存在则ok == true，否则ok为false
```

删除键值
```
delete(map1, key1)
```
for-range循环

关心key,value
```

for key, value := range map1 {
	...
}
```
只关心value
```
for _, value := range map1 {
	...
}
````
只关心key
```
for key := range map1 {
	...
}
```

**map 类型的切片**

切片每个元素是一个map，在go中需要定义两次,`我们必须使用两次 make() 函数，第一次分配切片，第二次分配 切片中每个 map 元素
`
```
	items := make([]map[int]int, 5)
	for i:= range items {
		items[i] = make(map[int]int, 1)
		items[i][1] = 2
	}
	这上面都是在初始化
	fmt.Printf("Version A: Value of items: %v\n", items)
```
**map 的排序**

`map 默认是无序的，不管是按照 key 还是按照 value 默认都不排序`

**思路**：转换成`slice`，然后利用`sort`排序

### 常用包
像`fmt`，`os`这种内置在`go`安装包里面的包，称之为标准库，有`150多个`
#### regex 包
变量 ok 将返回 true 或者 false,可以使用`MatchString`
```
ok, _ := regexp.MatchString(pat, searchIn)
```

#### 锁和 sync 包
当多个线程需要同时访问某个共享变量时，就可能出现问题，一个经典的解决方法是：`一次只能让一个线程对共享变量进行操作` 在go中通过`sync`包来实现

`sync.Mutex` 是一个互斥锁，它的作用是守护在临界区入口来确保同一时间只能有一个线程进入临界

```
import  "sync"

type Info struct {
	mu sync.Mutex
	// ... other fields, e.g.: Str string
}
func Update(info *Info) {
	info.mu.Lock()  上锁
    // critical section:
    info.Str = // new value
    // end critical section
    info.mu.Unlock() 释放锁
}
```
在 sync 包中还有一个 `RWMutex` 锁,他能通过 `RLock()` 来允许同一时间多个线程对变量进行读操作，但是只能一个线程进行写操作.

但上锁会影响性能问题，所以很多时候，我们会考虑使用`goroutines`和`channel`的方式来解决

#### big包
用于表示精密计算的数据

#### 导入包
导入包的格式为：

`import "包的路径或 URL 地址" `

如果是同一个项目中的包，则使用包的路径，这个路径是相对路径，如果是`remote package`的话，则使用`URL`地址

**导入本地自定义的包**
```
import (
	"fmt"
	"./pack1"
)
```
但这种方式不够好，因为调用方法的时候必须写` pack1.ReturnStr()`

更进一步 用别名`import with . :`
```
import . "./pack1"
```
就可以不用使用包名调用方法`test := ReturnStr()`

导入外部安装包`remote package`
1. `go install`先进行安装
```
go install codesite.ext/author/goExample/goex
```
2. 导入包 这里用`URL路径`
```
import goex "codesite.ext/author/goExample/goex"
```

`go install`  将包安装到本地它会从远端仓库下载包：下载源码、检出、编译和安装一气呵成。如果是本地项目的话，也会将源码安装在本地目录下
类似`python`中pip包管理工具

- `$GOPATH/src` go install的包源码存放在该目录下
- `$GOPATH/PKG/"machine_arch"/` 包被安装在该目录下

#### 远端包查询平台
Go Walker https://gowalker.org/

### 结构体
在什么情况下使用它？ 

当需要定义一个类型，它由一系列属性组成，**每个属性都有自己的类型和值的时候**，就应该使用结构体，它把数据聚集在一起

结构体也是值类型，可以通过`new()`来创建一个实例对象

通过`.`来访问字段属性

结构体的定义
```
type identifier struct {
    field1 type1
    field2 type2
    ...
}
```
三种不同的方式创建结构体对象 见main()
```
package main
import (
    "fmt"
    "strings"
)

type Person struct {
    firstName   string
    lastName    string
}

func upPerson(p *Person) {
    p.firstName = strings.ToUpper(p.firstName)
    p.lastName = strings.ToUpper(p.lastName)
}

func main() {
    // 1-struct as a value type:
    var pers1 Person
    pers1.firstName = "Chris"
    pers1.lastName = "Woodward"
    upPerson(&pers1)
    fmt.Printf("The name of the person is %s %s\n", pers1.firstName, pers1.lastName)

    // 2—struct as a pointer:
    pers2 := new(Person)
    pers2.firstName = "Chris"
    pers2.lastName = "Woodward"
    (*pers2).lastName = "Woodward"  // 这是合法的
    upPerson(pers2)
    fmt.Printf("The name of the person is %s %s\n", pers2.firstName, pers2.lastName)

    // 3—struct as a literal:
    pers3 := &Person{"Chris","Woodward"}
    upPerson(pers3)
    fmt.Printf("The name of the person is %s %s\n", pers3.firstName, pers3.lastName)
}
```
选择使用`new`的方式方便，由于`new()`返回的是指针，所以接收方法的参数需要使用定义`指针对象`来接收

#### 使用工厂方法创建结构体实例
go没有类概念，也就没有类方法来实现对结构体的操作，但我们可以根据可见性原则，迫使使用者**必须通过函数的方式创建结构体实例**

实例：
```
type matrix struct {
    ...
}

func NewMatrix(params) *matrix {
    m := new(matrix) // 初始化 m
    return m
}
```
在其他包中使用工厂方法：
```
package main
import "matrix"
...
wrong := new(matrix.matrix)     // 编译失败（matrix 是私有的）
right := matrix.NewMatrix(...)  // 实例化 matrix 的唯一方式
```

#### 带标签的结构体

结构体中的字段除了有名字和类型外，还可以有一个可选的标签（tag）
```
type TagType struct { // tags
	field1 bool   "An important answer"
	field2 string "The name of the thing"
	field3 int    "How much there are"
}
```
#### 方法
go语言中有一个`method`,类似其他语言类中定义的方法

Go 方法是作用在`接收者（receiver）`上的一个函数，接收者是`某种类型`的变量。因此方法是一种`特殊类型的函数`.

适用类型范围：**所有** 接收者类型可以是（几乎）任何类型，不仅仅是结构体类型,任何类型都可以有方法，甚至可以是函数类型

定义范围：**只要在同一个包中就可以**

定义格式：
```
func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }
```
不同的是`(recv receiver_type)`,接收者指明是哪个对象的方法

调用：**实例.方法()**

详细例子：
```
package main

import "fmt"

type TwoInts struct {
	a int
	b int
}

func main() {
	two1 := new(TwoInts)
	two1.a = 12
	two1.b = 10

	fmt.Printf("The sum is: %d\n", two1.AddThem())
	fmt.Printf("Add them to the param: %d\n", two1.AddToParam(20))

	two2 := TwoInts{3, 4}
	fmt.Printf("The sum is: %d\n", two2.AddThem())
}

func (tn *TwoInts) AddThem() int {
	return tn.a + tn.b
}

func (tn *TwoInts) AddToParam(param int) int {
	return tn.a + tn.b + param
}
```
`two1.AddThem()`通过实例对象，然后调用对应的方法`addThem()`

go通过这种注册的方式，实现了解耦合，并且`recevier`参数尽可能使用指针，因为**指针快** 如`func (tn *TwoInts)`

#### seter和getter方法
```
package person

type Person struct {
	firstName string  
	lastName  string
}

func (p *Person) FirstName() string {
	return p.firstName
}

func (p *Person) SetFirstName(newName string) {
	p.firstName = newName
}
```
use_person.go
```
package main

import (
	"./person"
	"fmt"
)

func main() {
	p := new(person.Person)
	// p.firstName undefined
	// (cannot refer to unexported field or method firstName)
	// p.firstName = "Eric"
	p.SetFirstName("Eric")
	fmt.Println(p.FirstName()) // Output: Eric
}
```
无法直接访问 因为是小写字母开头，于是通过getter setter的方式访问

#### 内嵌类型的方法和继承
这种方式就是`go`中的类继

当一个匿名类型被内嵌在结构体中时，匿名类型的可见方法也同样被内嵌，这在效果上等同于外层类型 继承 了这些方法：**将父类型放在子类型中来实现亚型**


```
type Engine interface {
	Start()
	Stop()
}

type Car struct {
	Engine
}
```
构建代码
```
func (c *Car) GoToWorkIn() {
	// get in car
	c.Start()
	// drive to work
	c.Stop()
	// get out of car
}
```

```
package main

import (
	"fmt"
	"math"
)

type Point struct {
	x, y float64
}

func (p *Point) Abs() float64 {
	return math.Sqrt(p.x*p.x + p.y*p.y)
}

type NamedPoint struct {
	Point
	name string
}

func main() {
	n := &NamedPoint{Point{3, 4}, "Pythagoras"}
	fmt.Println(n.Abs()) // 打印5
}
```
#### 垃圾回收
Go 开发者**不需要写代码来释放程序中不再使用的变量和结构占用的内存，在 Go运行时**中有一个独立的进程，即垃圾收集器（GC），会处理这些事情，它搜索不再使用的变量然后释放它们的内存。可以通过`runtime`包访问`GC`进程

通过调用 `runtime.GC()` 函数可以显式的触发 GC，但这**只在某些罕见的场景**下才有用

获取内存信息
```
var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("%d Kb\n", m.Alloc / 1024)
```





#### 方法总结

go并没有显示的类定义，但
**实际上通过定义变量加方法就可以实现出其他语言中类的概念**，对这就是go中的类(**数据和关联的方法**),Go 并不知道类似面向对象语言的类继承的概念。继承有两个好处：**代码复用和多态**,但都提供了其他的方式实现，
代码复用通过**组合和委托**实现，多态通过**接口**的使用来实现


### 接口
Go 语言不是一种 “传统” 的面向对象编程语言：它里面没有类和继承的概念.

但go语言的接口却**非常灵活**

go语言接口特性：
1. 接口很短，通常包含0个，最多3个方法；
2. 只包含一个方法的，按照约定，接口的名字由方法名加`[e]r`后缀组成，比如`Logger`,`Writer`
3. **接口被隐式地实现** 类型不需要显式声明它实现了某个接口
4. **多个类型可以实现同一个接口**
5. **一个类型可以实现多个接口**
6. **实现某个接口的类型（除了实现接口方法外）可以有其他的方法，但所有接口方法必须实现**


在go中，接口的功能拥有与其他语言的继承是一样的功能。

第一个例子：
```
package main

import "fmt"

type Shaper interface {
	Area() float32
}

type Square struct {
	side float32
}

func (sq *Square) Area() float32 {
	return sq.side * sq.side
}

func main() {
	sq1 := new(Square)
	sq1.side = 5

	var areaIntf Shaper
	areaIntf = sq1 //Square 变量赋值给了接口类型的变量
	// shorter,without separate declaration:
	// areaIntf := Shaper(sq1)
	// or even:
	// areaIntf := sq1
	fmt.Printf("The square has area: %f\n", areaIntf.Area())
}
```
程序解释：
- 定义了一个结构体 `Square`和一个接口`Shaper`，接口有一个方法 `Area()`
- `Square`实现了接口`Shaper`的方法`Area()`
- 可以将一个`Square` 类型的变量赋值给一个接口类型的变量：`areaIntf = sq1` 这一步实现了多态

更能体现多态的例子，**同样的方法表现出不同的行为**
```
package main

import "fmt"

type Shaper interface {
	Area() float32
}

type Square struct {
	side float32
}

func (sq *Square) Area() float32 {
	return sq.side * sq.side
}

type Rectangle struct {
	length, width float32
}

func (r Rectangle) Area() float32 {
	return r.length * r.width
}

func main() {

	r := Rectangle{5, 3} // Area() of Rectangle needs a value
	q := &Square{5}      // Area() of Square needs a pointer
	// shapes := []Shaper{Shaper(r), Shaper(q)}
	// or shorter
	shapes := []Shaper{r, q}
	fmt.Println("Looping through shapes for area ...")
	for n, _ := range shapes {
		fmt.Println("Shape details: ", shapes[n])
		fmt.Println("Area of this shape is: ", shapes[n].Area())
	}
}
```


#### 接口嵌套
一个接口可以包含一个或多个其他的接口，这相当于直接将这些内嵌接口的方法列举在外层接口中一样

比如接口 `File` 包含了 `ReadWrite` 和 `Lock` 的所有方法，它还额外有一个 `Close()` 方法
```
type ReadWrite interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
}

type Lock interface {
    Lock()
    Unlock()
}

type File interface {
    ReadWrite
    Lock
    Close()
}
```
#### 类型判断

由于接口的动态属性，一个接口的变量可以包含任意类型的值，那如何知道当前该接口具体实现的类型是哪个啦？

可以通过类型断言的方式 
```
if v, ok := varI.(T); ok {  // checked type assertion
    Process(v)
    return
}
// varI is not of type T

// 只关心ok
if _, ok := varI.(T); ok {
    // ...
}
```
`varI`是接口变量  `T`是实现类型

eg：
```
package main

import (
	"fmt"
	"math"
)

type Square struct {
	side float32
}

type Circle struct {
	radius float32
}

type Shaper interface {
	Area() float32
}

func main() {
	var areaIntf Shaper
	sq1 := new(Square)
	sq1.side = 5

	areaIntf = sq1
	// Is Square the type of areaIntf?
	if t, ok := areaIntf.(*Square); ok {
		fmt.Printf("The type of areaIntf is: %T\n", t)
	}
	if u, ok := areaIntf.(*Circle); ok {
		fmt.Printf("The type of areaIntf is: %T\n", u)
	} else {
		fmt.Println("areaIntf does not contain a variable of type Circle")
	}
}

func (sq *Square) Area() float32 {
	return sq.side * sq.side
}

func (ci *Circle) Area() float32 {
	return ci.radius * ci.radius * math.Pi
}
```

**方法二** 通过`type-switch`来实现
```
switch t := areaIntf.(type) {
case *Square:
	fmt.Printf("Type Square %T with value %v\n", t, t)
case *Circle:
	fmt.Printf("Type Circle %T with value %v\n", t, t)
case nil:
	fmt.Printf("nil value: nothing to check?\n")
default:
	fmt.Printf("Unexpected type %T\n", t)
}

输出：
Type Square *main.Square with value &{5}
```

#### 测试某个值是否实现了某个接口
```
type Stringer interface {
    String() string
}

if sv, ok := v.(Stringer); ok {
    fmt.Printf("v implements String(): %s\n", sv.String()) // note: sv, not v
}
```

#### 使用方法集与接口

作用于变量上的方法实际上是不区分变量到底是指针还是值的,但在某种情况下会是例外，即碰到的是**接口类型值**，就会变得有点复杂

其中`a Appender` 就是接口类型值
```
type Appender interface {
	Append(int)
}

func CountInto(a Appender, start, end int) {
	for i := start; i <= end; i++ {
		a.Append(i)
	}
}
```

在接口上调用方法时，需遵循以下原则：

- 指针方法可以通过指针调用
-值方法可以通过值调用
- 接收者是值的方法可以通过指针调用，因为指针会首先被解引用
- 接收者是指针的方法不可以通过值调用，因为存储在接口中的值没有地址

#### 空接口

空接口：不包含任意方法，它对实现不做任何要求
```
type Any interface {}
```
特点：
- 任何类型都实现了空接口
- 可以给空接口类型的变量赋予任何类型的值
- 这类似object对象


空接口有很多好处，比如构造通用类型的接口
```
type Element interface{}

type Vector struct {
	a []Element
}

由于Element可以是任意类型，所以这里的Vector类实际上就是包含了任意类型的slice

```

#### 反射

**反射：** 是元编程中的一种形式，可是在程序运行时检查类型和变量，很强大，但除非必要，尽量少用。

反射包常用的两个方法：

- `reflect.TypeOf` 返回被检查对象的类型
- `reflect.ValueOf` 返回被检查对象的值

其中`v := eflect.ValueOf`的变量有更多的属性

`v.kind`返回底层的数据类型

`v.interface` 返回值

比如
```
// blog: Laws of Reflection
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var x float64 = 3.4
	fmt.Println("type:", reflect.TypeOf(x))
	v := reflect.ValueOf(x)
	fmt.Println("value:", v)
	fmt.Println("type:", v.Type())  //类型
	fmt.Println("kind:", v.Kind()) // 返回底层类型
	fmt.Println("value:", v.Float()) 
	fmt.Println(v.Interface())
	fmt.Printf("value is %5.2e\n", v.Interface()) 
	y := v.Interface().(float64)
	fmt.Println(y)
}

```
output:
```
type: float64
value: 3.4
type: float64
kind: float64
value: 3.4
3.4
value is 3.40e+00
3.4
```

通过反射可以修改值
```
var x float64 = 3.4
v := reflect.ValueOf(&x) //这里必须取地址
v = v.Elem()
v.SetFloat(3.1415) // this works!
```

通过**反射获取类型所有的变量**
1. `NumField()` 方法返回结构内的字段数量；通过一个 `for` 循环用索引取得每个字段的值 `Field(i)`
2. 或者通过这种方法 `Method(n).Call(nil)`
```
func main() {
	value := reflect.ValueOf(secret) // <main.NotknownType Value>
	// iterate through the fields of the struct:
	for i := 0; i < value.NumField(); i++ {
		fmt.Printf("Field %d: %v\n", i, value.Field(i))
	}

	// call the first method, which is String():
	results := value.Method(0).Call(nil)
	fmt.Println(results) // [Ada - Go - Oberon]
}
```

**go语言面向对象总结**

Go 没有类，而是通过松耦合的**类型、方法对接口的实现**

OO 语言最重要的三个方面分别是：封装，继承和多态，在 Go 中它们是怎样表现的呢？

- 封装（数据隐藏）：和别的 OO 语言有 4 个或更多的访问层次相比，Go 把它简化为了 2 层
    - **包范围内的**：通过标识符首字母小写，对象 只在它所在的包内可见
    - **可导出的**：通过标识符首字母大写，对象对所在包以外也可见

类型只拥有自己所在包中定义的方法
- **继承：** 用组合实现：内嵌一个（或多个）包含想要的行为（字段和方法）的类型；多重继承可以通过内嵌多个类型实现
- **多态：** 用接口实现：某个类型的实例可以赋给它所实现的任意接口类型的变量。类型和接口是松耦合的，并且多重继承可以通过实现多个接口实现。Go 接口不是 Java 和 C# 接口的变体，而且接口间是不相关的，并且是大规模编程和可适应的演进型设计的关键


#### 结构体 集合 高阶函数

通常你在应用中定义了一个结构体，那么你也可能需要这个结构体的（指针）对象集合

```
type Any interface{}
type Car struct {
	Model        string
	Manufacturer string
	BuildYear    int
	// ...
}

type Cars []*Car  //指针集合
```
1. 定义一个通用的`Process()` 函数，它接收一个作用于**每一辆 car 的 f 函数作参数**
```
// Process all cars with the given function f:
func (cs Cars) Process(f func(car *Car)) {
	for _, c := range cs {
		f(c)
	}
}
```
2. 实现一个查找函数来获取子集合，并在`Process()`中传入一个闭包执行
```
// Find all cars matching a given criteria.
func (cs Cars) FindAll(f func(car *Car) bool) Cars {

	cars := make([]*Car, 0)
	cs.Process(func(c *Car) {
		if f(c) {
			cars = append(cars, c)
		}
	})
	return cars
}
```

### 读写数据
#### 读取用户输入
方法一：使用`fmt`包中`Scanln`和`Scanf`的方法
方法二：利用`bufio`中的读缓冲来读取数据
```
package main
import (
    "fmt"
    "bufio"
    "os"
)

var inputReader *bufio.Reader
var input string
var err error

func main() {
    inputReader = bufio.NewReader(os.Stdin) //创建一个读取器，与标准输入绑定
    fmt.Println("Please enter some input: ")
    input, err = inputReader.ReadString('\n')
    if err == nil {
        fmt.Printf("The input was: %s\n", input)
    }
}
```
#### 读取文件
```
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
)

func main() {
    inputFile, inputError := os.Open("input.dat")
    if inputError != nil {
        fmt.Printf("An error occurred on opening the inputfile\n" +
            "Does the file exist?\n" +
            "Have you got acces to it?\n")
        return // exit the function on error
    }
    defer inputFile.Close()

    inputReader := bufio.NewReader(inputFile)
    for {
        inputString, readerError := inputReader.ReadString('\n')
        fmt.Printf("The input was: %s", inputString)
        if readerError == io.EOF {
            return
        }      
    }
}
```
步骤：
1. `os.File`类型来打开文件
2. `bufio`创建读写器
3. `读写器读写文件内容` `inputReader.ReadString('\n')`
4. 判断文件结束`io.EOF`
5. 关闭文件 `defer`

-  将**整个文件的内容**读到一个字符串里：
   可以使用 `io/ioutil` 包里的 `ioutil.ReadFile()`方法
- 读取非字符串的文件 不能使用`ReadString()`, 使用`bufio.Reader` 的 `Read()`
```
buf := make([]byte, 1024)
...
n, err := inputReader.Read(buf)
if (n == 0) { break}

变量`n`的值表示读取到的字节数
```

**写文件**
```
package main

import (
	"os"
	"bufio"
	"fmt"
)

func main () {
	// var outputWriter *bufio.Writer
	// var outputFile *os.File
	// var outputError os.Error
	// var outputString string
	outputFile, outputError := os.OpenFile("output.dat", os.O_WRONLY|os.O_CREATE, 0666)
	if outputError != nil {
		fmt.Printf("An error occurred with file opening or creation\n")
		return  
	}
	defer outputFile.Close()

	outputWriter := bufio.NewWriter(outputFile)
	outputString := "hello world!\n"

	for i:=0; i<10; i++ {
		outputWriter.WriteString(outputString)
	}
	outputWriter.Flush()
}
```

#### 命令行读取参数
1. `os.Args` 处理基本的命令行参数
2. `flag`是扩展的命令行解析项，更多使用这种


#### json格式序列化

**序列化：** 将数据格式转换成字符串，用于传输

**反序列化：**
将字符串转换成对应的格式

`json.Marshal()` 实现序列化 `json.UnMarshal()`实现反序列化

json 包使用 `map[string]interface{}` 和 `[]interface{}` 储存任意的`JSON 对象和数组`；其可以被反序列化为任何的 JSON blob 存储到接口值中

json序列化并写入文件
```
// json.go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
)

type Address struct {
	Type    string
	City    string
	Country string
}

type VCard struct {
	FirstName string
	LastName  string
	Addresses []*Address
	Remark    string
}

func main() {
	pa := &Address{"private", "Aartselaar", "Belgium"}
	wa := &Address{"work", "Boom", "Belgium"}
	vc := VCard{"Jan", "Kersschot", []*Address{pa, wa}, "none"}
	// fmt.Printf("%v: \n", vc) // {Jan Kersschot [0x126d2b80 0x126d2be0] none}:
	// JSON format:
	js, _ := json.Marshal(vc)
	fmt.Printf("JSON format: %s", js)
	// using an encoder:
	file, _ := os.OpenFile("vcard.json", os.O_CREATE|os.O_WRONLY, 0666)
	defer file.Close()
	enc := json.NewEncoder(file)
	err := enc.Encode(vc)
	if err != nil {
		log.Println("Error in encoding json")
	}
}
```

#### go中的安全
- hash 包：实现了 `adler32、crc32、crc64 和 fnv` 校验；
- crypto 包：实现了其它的 `hash` 算法，比如 `md4、md5、sha1` 等。以及完整地实现了 `aes、blowfish、rc4、rsa、xtea` 等加密算法


#### 错误处理
go并没有其他语言那样的`try-catch`处理机制

错误有两类：
1. **预知的错误**，可以通过`error.new()` 返回错误值，调用处对错误值进行处理
2. `runtime`错误，**无法预知的错误**，这个通过`defer-panic-and-recover`机制解决

**预定义接口**
```
type error interface {
	Error() string
}
```

**定义一个错误**
```
err := errors.New("math - square root of negative number")
```

返回错误案例：
```
func Sqrt(f float64) (float64, error) {
	if f < 0 {
		return 0, errors.New ("math - square root of negative number")
	}
   // implementation of Sqrt
}

if f, err := Sqrt(-1); err != nil {
	fmt.Printf("Error: %s\n", err)
}

```
会打印出错误信息` “Error: math - square root of negative number”` 会打印出来。通常（错误信息）都会有像 `“Error:” `这样的前缀

**创建错误对象第二种方法：** 用 **fmt**创建错误对象，可以做一些格式化 
```
if f < 0 {
	return 0, fmt.Errorf("math: square root of negative number %g", f)
}
```

**自定义错误结构类型**：添加除错误信息以外更多有用的信息

```
// PathError records an error and the operation and file path that caused it.
type PathError struct {
	Op string    // "open", "unlink", etc.
	Path string  // The associated file.
	Err error  // Returned by the system call.
}

func (e *PathError) Error() string {
	return e.Op + " " + e.Path + ": "+ e.Err.Error()
}
```

**程序未知的运行时错误**

使用`defer-panic-and-recover`机制来处理 等价于其他语言的`try-catch`
1. `Go`运行时会触发运行时 `panic`，伴随着程序的崩溃抛出一个 `runtime.Error`接口类型的值
2. `recover`只能在`defer` 修饰的函数中使用：用于取得`panic` 调用中传递过来的错误值，如果是正常执行，调用 `recover`会返回 `nil`，且没有其它效果
3. `panic`会导致栈被展开直到 `defer`修饰的`recover()` 被调用或者程序中止
4. 我们可以显示的调用`panic`

只有`panic`的情况
```
package main

import "fmt"

func main() {
	fmt.Println("Starting the program")
	panic("A severe error occurred: stopping the program!")
	fmt.Println("Ending the program")
}
```
输出如下：
```
Starting the program
panic: A severe error occurred: stopping the program!
panic PC=0x4f3038
runtime.panic+0x99 /go/src/pkg/runtime/proc.c:1032
       runtime.panic(0x442938, 0x4f08e8)
main.main+0xa5 E:/Go/GoBoek/code examples/chapter 13/panic.go:8
       main.main()
runtime.mainstart+0xf 386/asm.s:84
       runtime.mainstart()
runtime.goexit /go/src/pkg/runtime/proc.c:148
       runtime.goexit()
---- Error run E:/Go/GoBoek/code examples/chapter 13/panic.exe with code Crashed
---- Program exited with code -1073741783
```
**panic,defer和recover** 结合的例子
```
// panic_recover.go
package main

import (
	"fmt"
)

func badCall() {
	panic("bad end")
}

func test() {
	defer func() {
		if e := recover(); e != nil {
			fmt.Printf("Panicing %s\r\n", e)
		}
	}()
	badCall()
	fmt.Printf("After bad call\r\n") // <-- wordt niet bereikt
}

func main() {
	fmt.Printf("Calling test\r\n")
	test()
	fmt.Printf("Test completed\r\n")
}
```
输出如下：
```
Calling test
Panicing bad end  
Test completed
```

### 单元测试
1. go中的测试工具是 `go test`
2. 测试代码和包中的业务代码是**分开的**
3. 测试文件名满足这种形式 `*_test.go`
4. `_test`程序不会被普通的 Go 编译器编译,只有`go test` 会编译所有的程序：普通程序和测试程序
5. 测试文件中必须导入 `"testing"` 包
6. 并写一些名字以`TestZzz` 打头的全局函数，这里的`Zzz` 是被测试函数的字母描述
```
func TestAbcde(t *testing.T)
```

一般来说，在函数的结尾把输出跟想要的结果对比，如果不等就打印一个错误，成功的测试则直接返回，有几种函数通知失败：
1. `func (t *T) Fail()` 标记测试函数为失败，然后继续执行（剩下的测试）
2. `func (t *T) FailNow()` 标记测试函数为失败并中止执行；文件中别的测试也被略过，继续执行下一个文件。
3. `func (t *T) Log(args ...interface{})`args 被用默认的格式格式化并打印到错误日志中。
4. `func (t *T) Fatal(args ...interface{})`结合 先执行 3），然后执行 2）的效果。

简单测试例子：

功能代码：
```
package even

func Even(i int) bool {		// Exported function
	return i%2 == 0
}

func Odd(i int) bool {		// Exported function
	return i%2 != 0
}
```
测试代码：
```
package even

import "testing"

func TestEven(t *testing.T) {
	if !Even(10) {
		t.Log(" 10 must be even!")
		t.Fail()
	}
	if Even(7) {
		t.Log(" 7 is not even!")
		t.Fail()
	}

}

func TestOdd(t *testing.T) {
	if !Odd(11) {
		t.Log(" 11 must be odd!")
		t.Fail()
	}
	if Odd(10) {
		t.Log(" 10 is not odd!")
		t.Fail()
	}
}
```

同时输入测试数据和结果数据，然后做对比
```
var tests = []struct{ 	// Test table
	in  string
	out string

}{
	{"in1", "exp1"},
	{"in2", "exp2"},
	{"in3", "exp3"},
...
}

func TestFunction(t *testing.T) {
	for i, tt := range tests {
		s := FuncToBeTested(tt.in)
		if s != tt.out {
			t.Errorf("%d. %q => %q, wanted: %q", i, tt.in, s, tt.out)
		}
	}
}
```

### 协程和通道

多线程并发领域，最主要的问题是线程同步(**数据共享**)，传统的解决方法是通过加锁的方式，但这种方式会导致性能低下，业务复杂。

Go语言并不推荐这种方式，更推荐使用**通道**的方式进行数据的同步(FIFO)，一种消息队列的机制。

Go也不使用线程来进行并发，反而使用**协程goroutines**的方式来进行并发，它比线程更轻量级，开销更小。

当系统调用（比如等待 I/O）**阻塞协程**时，其他协程会继续在**其他线程**上工作。协程的设计隐藏了许多线程创建和管理方面的复杂工作。

任何 `Go`程序都必须有`main()` 函数也可以看做是一个协程，尽管它并没有通过`go`来启动，也可以在初始化`init`中启动

当`main()` 函数返回的时候，程序退出：它不会等待任何其他非`main`协程的结束



**并发和并行**

go的重点并不是在于并行，所以默认是并发执行，如果想实现并行的效果，也只需要指定`GOMAXPROCS`变量就可以，这时候go的协程就是并行执行了，协程会平均分配到处理器中

假设`n`是机器上处理器或者核心的数量。如果你设置环境变量 `GOMAXPROCS>=n`，或者执行 `runtime.GOMAXPROCS(n)`，接下来协程会被分割（分散）到`n` 个处理器上。

更多的处理器并不意味着性能的线性提升。有这样一个经验法则，对于 n 个核心的情况设置`GOMAXPROCS`为 `n-1`以获得最佳性能

也同样需要遵守这条规则：`协程的数量 > 1 + GOMAXPROCS > 1`

命令行指定并行数量,只需要这一条
`runtime.GOMAXPROCS(*numCores)`

如何开启一个协程？
在方法或者函数调用时 加上`go`,这个函数就会新开
一个协程运行。

**Go 协程 与其他语言的协程对比**
- Go **协程意味着并行**（或者可以以并行的方式部署），协程一般来说不是这样的
- Go 协程通过**通道**来通信；协程通过让出和恢复操作来通信

Go 协程比协程更强大，也很容易从协程的逻辑复用到 Go 协程

#### 通道
很多时候多个协程之间需要进行通信？

传统的做法是通过共享内存变量来实现，而Go的实现思路是通过`channel通道`来实现，通道天然是同步的。

通道的定义格式：
`var identifier chan datatype`
比如：
```
var ch1 chan string
ch1 = make(chan string)
```
更常用简短的方式：
```
ch1 := make(chan string)
```

**通信操作符 <-**
- 写入通道 `ch <- int1`
- 从通道中读数 `int2 := <- ch`、

特殊用法`<- ch` 获取**通道的值，当前值会被丢弃**
```
if <- ch != 1000{
	...
}
```

#### 通道阻塞
默认情况下，通道是**同步无缓冲的**
1. 对于同一个通道，对于发送方来说，在接收者准备好之前是阻塞，
2. 对于同一个通道，接收操作是阻塞的（协程或函数中的），直到发送者可用：如果通道中没有数据，**接收者就阻塞了**

无缓冲通道 只能包含**一个元素**
```
out := make(chan int)
```
缓冲通道 可以容纳**多个元素**
```
buf := 100
ch1 := make(chan string, buf)
```
如果容量大于 0，通道就是异步的了：缓冲满载（发送）或变空（接收）之前通信不会阻塞，元素会按照发送的顺序被接收

在设计算法时首先考虑使用**无缓冲通道**，只在不确定的情况**下使用缓冲**

**信号量的模式**
协程通过在通道`ch` 中放置一个值来处理结束的信号,通过这种方式来进行分发任务处理

举例：
```
type Empty interface {}
var empty Empty
...
data := make([]float64, N)
res := make([]float64, N)
sem := make(chan Empty, N)
...
for i, xi := range data {
	go func (i int, xi float64) {
		res[i] = doSomething(i, xi)
		sem <- empty
	} (i, xi)
}
// wait for goroutines to finish
for i := 0; i < N; i++ { <-sem }
```

将任务进行分解，每当有任务完成时，就往`sem`通道里面写值，而`for`循环会获取到对应完成信号的个数，如果没有获取到会阻塞，直到获取到为止。

#### 使用for循环启动协程
```
for i, v := range data {
	go func (i int, v float64) {
		doSomething(i, v)
		...
	} (i, v)
}
```
#### for循环读取chanel
```
for v := range ch {
	fmt.Printf("The value is %v\n", v)
}
```

#### 关闭通道
通道可以被显式的关闭；尽管它们和文件不同：不必每次都关闭。**只有在当需要告诉接收者不会再提供新的值的时候，才需要关闭通道**。只有发送者需要关闭通道，**接收者永远不会需要**

发送方通过`defer`关闭通道
```
ch := make(chan float64)
defer close(ch)
```

接收方如何判断通道关闭了？

```
v, ok := <-ch
if !ok {
  break
}
process(v)

// eg：

for {
		input, open := <-ch
		if !open {
			break
		}
		fmt.Printf("%s ", input)
	}
```

#### 使用select监听切换协程
```
select {
case u:= <- ch1:
        ...
case v:= <- ch2:
        ...
        ...
default: // no value ready to be received
        ...
}
```
其中`default`不是必须的，select监听进入通道的数据，也可以是用通道发送值的时候

**select处理逻辑**：
- 如果都阻塞了，会等待直到其中一个可以处理
- 如果多个可以处理，随机选择一个
如果没有通道操作可以处理并且写了
- default语句，它就会执行：default 永远是可运行的（这就是准备好了，可以执行）。

**使用场景**

select 语句实现了一种监听模式，通常用在**（无限）循环**中；在某种情况下，通过`break`语句使循环退出

```
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go pump1(ch1)
	go pump2(ch2)
	go suck(ch1, ch2)

	time.Sleep(1e9)
}

func pump1(ch chan int) {
	for i := 0; ; i++ {
		ch <- i * 2
	}
}

func pump2(ch chan int) {
	for i := 0; ; i++ {
		ch <- i + 5
	}
}

func suck(ch1, ch2 chan int) {
	for {
		select {
		case v := <-ch1:
			fmt.Printf("Received on channel 1: %d\n", v)
		case v := <-ch2:
			fmt.Printf("Received on channel 2: %d\n", v)
		}
	}
}
```

#### worker模式
主线程
```
    func main() {
        pending, done := make(chan *Task), make(chan *Task)
        go sendWork(pending)       // put tasks with work on the channel
        for i := 0; i < N; i++ {   // start N goroutines to do work
            go Worker(pending, done)
        }
        consumeWork(done)          // continue with the processed tasks
    }
```
worker
```
    func Worker(in, out chan *Task) {
        for {
            t := <-in
            process(t)
            out <- t
        }
    }
```
主线程往pending通道里面放任务，而work线程专门从pending里面拿任务并且执行

#### 限制请求数量

缓冲队列的容量大小就是同时能够处理最大的数量

```
package main

const MAXREQS = 50

var sem = make(chan int, MAXREQS)

type Request struct {
	a, b   int
	replyc chan int
}

func process(r *Request) {
	// do something
}

func handle(r *Request) {
	sem <- 1 // doesn't matter what we put in it
	process(r)
	<-sem // one empty place in the buffer: the next request can start
}

func server(service chan *Request) {
	for {
		request := <-service
		go handle(request)
	}
}

func main() {
	service := make(chan *Request)
	go server(service)
}
```

#### 协程在服务器端的应用

`server`协程会无限循环以从`chan *Request`接收请求，并且为了避免被长时间操作所堵塞，它将为每一个请求启动一个协程来做具体的工作

```
package main

import "fmt"

type Request struct {
	a, b   int
	replyc chan int // reply channel inside the Request
}

type binOp func(a, b int) int

func run(op binOp, req *Request) {
	req.replyc <- op(req.a, req.b)
}

func server(op binOp, service chan *Request) {
	for {
		req := <-service // requests arrive here
		// start goroutine for request:
		go run(op, req) // don't wait for op
	}
}

func startServer(op binOp) chan *Request {
	reqChan := make(chan *Request)
	go server(op, reqChan)
	return reqChan
}

func main() {
	adder := startServer(func(a, b int) int { return a + b })
	const N = 100
	var reqs [N]Request
	for i := 0; i < N; i++ {
		req := &reqs[i]
		req.a = i
		req.b = i + N
		req.replyc = make(chan int)
		adder <- req
	}
	// checks:
	for i := N - 1; i >= 0; i-- { // doesn't matter what order
		if <-reqs[i].replyc != N+2*i {
			fmt.Println("fail at", i)
		} else {
			fmt.Println("Request ", i, " is ok!")
		}
	}
	fmt.Println("done")
}
```

#### 多核心上并行执行
关键代码`runtime.GOMAXPROCS(NCPU)`
```
func DoAll(){
    sem := make(chan int, NCPU) // Buffering optional but sensible
    for i := 0; i < NCPU; i++ {
        go DoPart(sem)
    }
    // Drain the channel sem, waiting for NCPU tasks to complete
    for i := 0; i < NCPU; i++ {
        <-sem // wait for one task to complete
    }
    // All done.
}

func DoPart(sem chan int) {
    // do the part of the computation
    sem <-1 // signal that this piece is done
}

func main() {
    runtime.GOMAXPROCS(NCPU) // runtime.GOMAXPROCS = NCPU
    DoAll()
}
  
```
- DoAll()函数创建了一个sem通道，每个并行计算都将在对其发送完成信号；在一个 for 循环中NCPU个协程被启动了，每个协程会承担1/NCPU的工作量。每一个DoPart()协程都会向sem通道发送完成信号。
- DoAll()会在 for 循环中等待NCPU个协程完成：sem通道就像一个信号量，这份代码展示了一个经典的信号量模式


#### 对于流水线的场景

一个更高效的计算方式是**让每一个处理步骤作为一个协程独立工作**。每一个步骤从上一步的输出通道中获得输入数据

更改前
```
func SerialProcessData(in <-chan *Data, out chan<- *Data) {
    for data := range in {
        tmpA := PreprocessData(data)
        tmpB := ProcessStepA(tmpA)
        tmpC := ProcessStepB(tmpB)
        out <- PostProcessData(tmpC)
    }
}
```
更改后
```
func ParallelProcessData (in <-chan *Data, out chan<- *Data) {
    // make channels:
    preOut := make(chan *Data, 100)
    stepAOut := make(chan *Data, 100)
    stepBOut := make(chan *Data, 100)
    stepCOut := make(chan *Data, 100)
    // start parallel computations:
    go PreprocessData(in, preOut)
    go ProcessStepA(preOut,StepAOut)
    go ProcessStepB(StepAOut,StepBOut)
    go ProcessStepC(StepBOut,StepCOut)
    go PostProcessData(StepCOut,out)
}   
```

### 网络和应用

#### tcp服务器
`net`包处理`tcp` `udp`等协议

tcp服务器应用
```
package main

import (
	"fmt"
	"net"
)

func main() {
	fmt.Println("Starting the server ...")
	// 创建 listener
	listener, err := net.Listen("tcp", "localhost:50000")
	if err != nil {
		fmt.Println("Error listening", err.Error())
		return //终止程序
	}
	// 监听并接受来自客户端的连接
	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("Error accepting", err.Error())
			return // 终止程序
		}
		go doServerStuff(conn)
	}
}

func doServerStuff(conn net.Conn) {
	for {
		buf := make([]byte, 512)
		len, err := conn.Read(buf)
		if err != nil {
			fmt.Println("Error reading", err.Error())
			return //终止程序
		}
		fmt.Printf("Received data: %v", string(buf[:len]))
	}
}
```
每个连接都起**一个协程**来进行处理

#### 简单的Web服务器
web服务器是http应用，可通过浏览器的URL的方式直接访问

通过`net/http`包来实现该功能

- `http.URL` 表示网页地址
- `http.Request` 描述客户端请求
- `http.HandleFunc` 注册某个URL对应的处理函数

第一个参数是**请求的路径**，第二个参数是当路径被请求时，需要调用的**处理函数**的引用
```
http.HandleFunc("/", HelloServer)
```

web服务器简单案例
```
package main

import (
	"fmt"
	"log"
	"net/http"
)

func HelloServer(w http.ResponseWriter, req *http.Request) {
	fmt.Println("Inside HelloServer handler")
	fmt.Fprintf(w, "Hello,"+req.URL.Path[1:])
}

func main() {
	http.HandleFunc("/", HelloServer)
	err := http.ListenAndServe("localhost:8080", nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err.Error())
	}
}
```
然后打开浏览器并输入 url 地址：`http://localhost:8080/world`，浏览器就会出现文字：`Hello, world`

如果你需要使用安全的 https 连接，使用 `http.ListenAndServeTLS()` 代替 `http.ListenAndServe()`


#### 访问并读取网页
`http.Head()` 
`http.Get()` 

```
package main

import (
	"fmt"
	"net/http"
)

var urls = []string{
	"http://www.google.com/",
	"http://golang.org/",
	"http://blog.golang.org/",
}

func main() {
	// Execute an HTTP HEAD request for all url's
	// and returns the HTTP status string or an error string.
	for _, url := range urls {
		resp, err := http.Head(url)
		if err != nil {
			fmt.Println("Error:", url, err)
		}
		fmt.Println(url, ": ", resp.Status)
	}
}
```

#### 确保网页的健壮性

当网页应用的处理函数发生 panic，服务器会简单地终止运行。这可不妙：网页服务器必须是足够健壮的程序，能够承受任何可能的突发问题

`defer/recover`进行异常处理，但会产生大量的冗余代码，使用闭包的方式会更优雅。这种方式类似python中的`装饰器`

```
func logPanics(function HandleFnc) HandleFnc {
	return func(writer http.ResponseWriter, request *http.Request) {
		defer func() {
			if x := recover(); x != nil {
				log.Printf("[%v] caught panic: %v", request.RemoteAddr, x)
			}
		}()
		function(writer, request)
	}
}

http.HandleFunc("/test1", logPanics(SimpleServer))
http.HandleFunc("/test2", logPanics(FormServer))
```
### 常见错误
**1. 误用短声明：**

短声明是重新声明一个变量
```
var remember bool = false
if something {
    remember := true //错误
}
// 使用remember
```
外层`remember`永远都是false，起不到修改的作用，因为内层`remember`是重新声明的变量，需要将`remember := true`替换成`remember = true`

**2. 误用字符串**

字符串是常用，使用`a += b`实际上是新生成一个字符串，如果频繁使用，会导致大量的开销。

**应该使用一个字符数组代替字符串，将字符串内容写入一个缓存中**
```
var b bytes.Buffer
...
for condition {
    b.WriteString(str) // 将字符串str写入缓存buffer
}
    return b.String()
```
**3. make和new的使用**
- 切片、映射和通道，使用make
- 数组、结构体和所有的值类型，使用new 

**4. 指针的使用**
- slice本身就是指针，不要再声明指针变量
    正确的方式
    ```
    func findBiggest( listOfNumbers []int ) int {}
    ```
    错误的方式 注意`*`
    ```
     func findBiggest( listOfNumbers *[]int ) int {}
    ```
- 同理接口本身也是指针，不要在参数传递的声明指针变量
    `n nexter` 是对的
    ```
    type nexter interface {
        next() byte
    }
    func nextFew1(n nexter, num int) []byte {
        var b []byte
        for i:=0; i < num; i++ {
            b[i] = n.next()
        }
        return b
    }
    ```
    `n *nexter`是错的
    ```
    func nextFew2(n *nexter, num int) []byte {
        var b []byte
        for i:=0; i < num; i++ {
            b[i] = n.next() // 编译错误:n.next未定义（*nexter类型没有next成员或next方法）
        }
        return b
    }
   ```

- 使用**值类型**的时候尽量不要使用指针，因为这会给内存带来额外的开销

**总结** ： 指针类型的变量在申明时不要再申明成指针`*`，值类型的变量尽量不要使用指针的功能。

**5.误用协程和通道**

仅在代码中并发执行非常重要的时候，才使用协程和通道。

**6. 顺序和并发**
```
package main
import (
    "fmt"
    "time"
)

var values = [5]int{10, 11, 12, 13, 14}

func main() {
    // 版本A:
    for ix := range values { // ix是索引值
        func() {
            fmt.Print(ix, " ")
        }() // 调用闭包打印每个索引值
    }
    fmt.Println()
    // 版本B: 和A版本类似，但是通过调用闭包作为一个协程
    for ix := range values {
        go func() {
            fmt.Print(ix, " ")
        }()
    }
    fmt.Println()
    time.Sleep(5e9)
    // 版本C: 正确的处理方式
    for ix := range values {
        go func(ix interface{}) {
            fmt.Print(ix, " ")
        }(ix)
    }
    fmt.Println()
    time.Sleep(5e9)
    // 版本D: 输出值:
    for ix := range values {
        val := values[ix]
        go func() {
            fmt.Print(val, " ")
        }()
    }
    time.Sleep(1e9)
}
```

### 模式
#### ok模式
1. 检查`map`中是否存在某个`key`?
```
if value, isPresent = map1[key1]; isPresent {
        Process(value)
}
// key1不存在
…
```
2. 检查一个接口变量是否包含某个类型T ？
```
if value, ok := varI.(T); ok {
    Process(value)
}
// 接口类型varI没有包含类型T
```
3. 检查函数返回是否存在错误？
```
value, err := pack1.Func1(param1)

if err != nil {
    fmt.Printf("Error %s in pack1.Func1 with parameter %v", err.Error(), param1)
    return err
}
```
4. 检查通道是否关闭？
```
    for {
        if input, open := <-ch; !open {
            break // 通道是关闭的
        }
        Process(input)
    }
```

#### defer模式
defer的应用场景有两个
- 使用 defer 可以确保资源不再需要时，都会被恰当地关闭
- 它可以恢复 panic
1. 关闭文件
```
// 先打开一个文件 f
defer f.Close()
```
2. 释放锁
```
mu.Lock()
defer mu.Unlock()
```
3. 从panic 恢复
```
defer func() {
	if err := recover(); err != nil {
		log.Printf("run time panic: %v", err)
	}
}()
```

### 出于性能考虑的实用代码片段
#### 字符串操作
1. 获取子串
```
substr := str[n:m]
```
2. 使用for range 遍历
```
// gives the Unicode characters:
for ix, ch := range str {
…
}
```
3. 获取字符串的字符数
```
utf8.RuneCountInString(str)
```
4. 字符串拼接
```
最快速： `with a bytes.Buffer`

`Strings.Join()`

使用`+=`
```
#### 数组切片map
1. 创建
```
arr1 := new([len]type)
slice1 := make([]type, len)
map1 := make(map[keytype]valuetype)
```
2. 初始化
```
arr1 := [...]type{i1, i2, i3, i4, i5}
slice1 := []type{i1, i2, i3, i4, i5}
map1 := map[string]int{"one": 1, "two": 2}
```
3. 检查key是否存在
```
val1, isPresent = map1[key1]
```

4. 删除key
```
delete(map1, key1)
```

#### 结构体
1. 创建
```
type struct1 struct {
    field1 type1
    field2 type2
    …
}
ms := new(struct1)
```
2. 初始化
```
ms := &struct1{10, 15.5, "Chris"}
```

#### 接口分类匹配

```
func classifier(items ...interface{}) {
    for i, x := range items {
        switch x.(type) {
        case bool:
            fmt.Printf("param #%d is a bool\n", i)
        case float64:
            fmt.Printf("param #%d is a float64\n", i)
        case int, int64:
            fmt.Printf("param #%d is an int\n", i)
        case nil:
            fmt.Printf("param #%d is nil\n", i)
        case string:
            fmt.Printf("param #%d is a string\n", i)
        default:
            fmt.Printf("param #%d’s type is unknown\n", i)
        }
    }
}
```

#### 程序出错时终止程序
```
if err != nil {
   fmt.Printf("Program stopping with error %v", err)
   os.Exit(1)
}
```
or
```
if err != nil { 
	panic("ERROR occurred: " + err.Error())
}
```

#### 协程和通道
[协程与通道](https://github.com/unknwon/the-way-to-go_ZH_CN/blob/master/eBook/18.8.md)


#### 其他性能建议
1）尽可能的使用:=去初始化声明一个变量（在函数内部）；

2）尽可能的使用字符代替字符串；

3）尽可能的使用切片代替数组；

4）初始化映射时指定其容量；

5）当定义一个方法时，**使用指针类型作为方法的接受者；**

6）在代码中使用常量或者标志提取常量的值；









