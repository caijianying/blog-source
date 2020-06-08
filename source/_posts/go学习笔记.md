---
title: go学习笔记
tags:
  - 笔记
originContent: ''
categories:
  - go
toc: false
date: 2020-06-08 18:24:51
---

## 语法
### 特殊声明
#### _用法
- 用在import `_ "github.com/go-sql-driver/mysql"`

   程序默认执行`init`方法
- 用在函数返回值 `_, err := client.Do(req)`

  忽略相应的返回值
  
#### new函数
内建的`new`函数也是一种创建变量的方法，`new(type)`表示创建一个`type`类型的匿名变量，并初始化为`type`类型的零值，返回变量的地址，指针类型为`*type`。

```
p := new(int)   	// p, *int 类型, 指向匿名的 int 变量
fmt.Println(*p) 	// 0
*p = 2          	// 设置 int 匿名变量的值为 2
fmt.Println(*p) 	// 2
```

如下函数完成同样的功能：创建变量，返回变量地址
```
func newA() *int {
    return new(int)
}
func newB() *int {
    var i int
    return &i
}
```

### 基本类型
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8 的别名

rune // int32 的别名
    // 表示一个 Unicode 码点

float32 float64

complex64 complex128
本例展示了几种类型的变量。 同导入语句一样，变量声明也可以“分组”成一个语法块。
### 命名返回值
没有参数的 return 语句返回已命名的返回值。也就是 **直接** 返回
```
func method() (x, y int) {
	x = 1
	y = 2
	return
}
func main() {
	fmt.Println(method()) // 1 2
}
```

### 短变量声明
在函数中，简洁赋值语句 := 可在类型明确的地方代替 var 声明。

函数外的每个语句都必须以关键字开始（var, func 等等），因此 := 结构不能在函数外使用。

### 指针
**指针声明**：`var p *int`

**指针赋值**：`var p *int = &i` or `pp := &i`

**空指针**：nil

**指针使用**：主要经过三个步骤：声明、赋值和访问指针指向的变量的值
```
var i int = 10
var p *int = &i
*p = 12
fmt.Println(p,*p)
```

### 结构体
```
type Vertex struct {
	X, Y int
}

var (
	v1 = Vertex{1, 2}  // 创建一个 Vertex 类型的结构体
	v2 = Vertex{X: 1}  // Y:0 被隐式地赋予
	v3 = Vertex{}      // X:0 Y:0
	p  = &Vertex{1, 2} // 创建一个 *Vertex 类型的结构体（指针）
)
```

### 数组
**数组声明及赋值**

两种方式：
```
// 第一种
var a [2]string
a[0] = "hello"
a[1] = "world"
// 第二种
aa :=[2]int{10,20}
```
**遍历**
```
    // 第一种
	for i, s := range a {
		fmt.Printf("%d : %s\n", i, s)
	}
    // 第二种
	for i := 0; i < len(aa); i++ {
		fmt.Printf("%d : %d\n", i, aa[i])
	}
```

### map
**map声明及赋值**
```
// 第一种
var m = make(map[string]string)
m["a"] = "A"
m["b"] = "B"
// 第二种
var m = map[string]string{
		"a":"A",
		"b":"B",
}
```

**遍历**
```

```

### slice 切片
切片并不存储任何数据，它只是描述了底层数组中的一段。

更改切片的元素会修改其底层数组中对应的元素。

与它共享底层数组的切片都会观测到这些修改。


**声明**
```
a := [5]int{1,2,3,4,5}

// 类型 []T 表示一个元素类型为 T 的切片
// 第一种 不指定具体大小
q := []int{2, 3, 5, 7, 11, 13}
// 第二种，类型自动推断
qq := a[0:2]
// 第三种，声明为切片
var s []int =  a[1:4]
// 第四种
aa := make([]int, 5)
```

**用法**

切片 s 的长度和容量可通过表达式 `len(s)` 和 `cap(s)` 来获取。
## 流程控制语句：for、if、else、switch 和 defer
### for
```
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
```
初始化语句和后置语句是可选的
```
	for ; sum < 1000; {
		sum += sum
	}
```
for 是 Go 中的 “while”
```
	for sum < 10 {
		sum += 1
	}
```
无限循环
```
for {
	}
```

### if
```
	if  sum==0 {
		fmt.Println(sum)
	}
```
同 for 一样， if 语句可以在条件表达式前执行一个简单的语句。**该语句声明的变量作用域仅在 if 之内**。
```
	if sum = 1 ; sum==1 {
		fmt.Println(sum)
	}
```

### defer
defer 语句会将函数推迟到外层函数返回之后执行。推迟调用的函数其参数会立即求值，但直到外层函数返回前该函数都不会被调用。 个人猜测用途是关闭资源，处理异常等。

推迟的函数调用会被压入一**个栈**中。当外层函数返回时，被推迟的函数会按照后进先出的顺序调用
```
func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}
```

## 测试
go默认有个轻量级测试框架，可以使用`go test`命令和`testing`包

创建一个文件，文件名以` _test.go`结尾,函数名为`TestXXX`,并且传递参数`(t *testing.T)`

例如：
```
package morestrings

import "testing"

func TestReverseRunes(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := ReverseRunes(c.in)
		if got != c.want {
			t.Errorf("ReverseRunes(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

然后执行命令`go test`

## 方法和接口
### 函数与方法区别
**方法在func关键字后是接收者而不是函数名**

1. 普通函数
   ```
   func function_name([parameter list]) [return_types] {
   函数体
   }
   ```
2. 方法(如struct方法)
   ```
   func (variable_name variable_data_type) method_name([parameter list]) [return_type]{
   /* 函数体*/
   }
   ```

**使用区别** 

函数: `function_name()`  函数有参数的话，必须保持类型一致，否则编译失败。

方法：`p.method_name()`其中p可以为指针，也可以为值（p为结构体的值）

方法的接受者可以为指针，也可以为值。例如：
```
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

	v := Vertex{3, 4}
	fmt.Println(v.Abs())
	
	p := &Vertex{4, 3}
	fmt.Println(p.Abs())
```

**使用指针接收者的原因有二：**

1. 方法能够修改其接收者指向的值。

2. 这样可以避免在每次调用方法时复制该值。若值的类型为大型结构体时，这样做会更加高效。

### 接口
如果一个类型实现了一个接口需要的**所有方法**，那么该类型就实现了这个接口。

```
type User struct {
	name string
	age  int
}

type I interface {
	Name()
	Age()
}

func (u User)Name() {
	fmt.Printf("Name:%v\n", u.name)
}

func (u User)Age() {
	fmt.Printf("Age:%v\n", u.age)
}

func main() {
	var u I = User{"truman",18}
	u.Name()
	u.Age()
}
```

#### 类型断言
类型断言 提供了访问接口值底层具体值的方式。`t := i.(T)`
```
	var a interface{} = 11
	s:= a.(int)
	fmt.Println(s) //11

	b,ok:= a.(string)
	fmt.Println(b,ok)// false
```

## 并发
使用`go f(a)`即可新建一个goroutine

### 信道
声明一个信道`c := make(chan int)`

带缓存的信道 `c := make(chan int,10)`

仅当信道的缓冲区填满后，向其发送数据时才会阻塞。当缓冲区为空时，接受方会阻塞。

通过channel可以实现唤醒线程，例如：

```
	go func() {
		data := "hello value:stop"
		fmt.Println(data)
		time.Sleep(1000000000 * 10)
		ch <- data
	}()
	<-ch
	fmt.Println("execute next business.")
```

#### range/close
只有发送者才能关闭信道
```
	ch := make(chan string)
	go func() {
		for i:= range ch {
			if  i=="hello value:5" {
				time.Sleep(1000000000*10)
			}
			fmt.Println(i)
		}
	}()

	for i := 0; i < 10; i++ {
		data := "hello value:"+fmt.Sprintf("%d", i)
		ch <- data
	}
	close(ch)
```

#### sync.Mutex 
互斥锁

实现从0加到1000
```
type Account struct {
     money int
	 mux sync.Mutex
}

func (a *Account)Inc()  {
	a.mux.Lock()
    a.money++
    fmt.Println(a.money)
    defer a.mux.Unlock()
}

func main() {
	a:=Account{money: 0}
	for i := 0; i <1000; i++ {
		go a.Inc()
	}
	time.Sleep(time.Second*10)
	fmt.Printf("acount money:%d\n", a.money)
}
```

## 学习资源
1. [https://tour.go-zh.org/](https://tour.go-zh.org/)
2. [How to Write Go Code](https://golang.org/doc/code.html)
3. [Go語言聖經（中文版）](https://wizardforcel.gitbooks.io/gopl-zh/content/)


## 参考
1. [https://golang.org/doc/code.html#Testing](https://golang.org/doc/code.html#Testing)