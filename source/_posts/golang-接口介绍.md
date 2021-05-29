title: Golang --- 接口介绍
author: _Tao
tags: []
categories:
  - Golang
date: 2020-05-23 19:26:00
---
Go语言的主要设计者之一罗布·派克（Rob Pike）曾经说过，如果只能选择一个Go语言的特 性移植到其他语言中，他会选择接口。

接口在Go语言有着至关重要的地位。如果说goroutine和channel是支撑起Go语言的并发模型 的基石，让Go语言在如今集群化与多核化的时代成为一道极为亮丽的风景，那么接口是Go语言 整个类型系统的基石，让Go语言在基础编程哲学的探索上达到前所未有的高度。

Go语言在编程哲学上是变革派，而不是改良派。这不是因为Go语言有goroutine和channel， 而更重要的是因为Go语言的类型系统，更是因为Go语言的接口。Go语言的编程哲学因为有接口 而趋近完美。

<!-- more -->


    接口只有方法声明，没有实现，也没有数据字段。
    接口可以匿名嵌入到其他接口。
    对象赋值给接口时，会发生拷贝。
    只有当接口存储的类型和对象都是nil时，接口等于nil。
    空接口可以接收任意的数据类型。
    一个类型可以实现多个接口。
    接口变量名习惯以 er 结尾。

<br/>

    接口类型是对其它类型行为的抽象和概括，接口类型不会和特定的实现细节绑定。
    Go接口独特在它是隐式实现的，这是指：一个结构体只要实现了接口要求的所有方法，我们就说这个结构体实现了该接口。


​    
### 接口语法
接口在现实世界也是有真实场景的，如同笔记本上都有USB插口，且不用担心这个插槽是为手机、U盘、平板哪一个准备的，因为笔记本的usb插槽和各种设备的厂家统一了USB的插槽规范。

    type 接口名 interface {
        method1(参数列表)返回值列表
        method2(参数列表)返回值列表
    }
    interface类型可以定义一组方法，且不需要实现这些方法！并且interface不能有任何变量。
    只要有一个变量类型，含有接口中的所有方法，就说这个变量类型实现了这个接口。

#### Go多态与接口
```go
    package main
    import "fmt"
    //定义一个Usb接口，且定义Usb功能方法
    type Usb interface {
        Start()
        Stop()
    }
    type Phone struct {
        Name  string
        Price int
    }
    //让手机Phone实现Usb接口的方法
    func (p Phone) Start() {
        fmt.Println("手机已连接USB,开始工作")
    }
    //必须实现接口所有的方法，少一个都报错  如下：Phone does not implement Usb (missing Stop method)
    func (p Phone) Stop() {
        fmt.Println("手机断开了USB，停止工作")
    }
    type IPad struct {
        Name  string
        Price int
    }
    func (p IPad) Start() {
        fmt.Println("ipad已经连接USB，开始工作")
    }
    func (p IPad) Stop() {
        fmt.Println("ipad断开了USB，停止工作")
    }
    //定义一个电脑结构体，这个结构体可以实现usb的接口
    type Computer struct {
    }
    //定义working方法，接收Usb接口类型变量
    //实现Usb接口声明的所有方法
    //这是一个多态的函数，调用同一个Working函数，不同的执行结果
    func (mbp Computer) Working(usb Usb) {
        //不同的usb实参，实现不同的功能
        //只要实现了usb接口的数据类型，那么这个类型的变量，就可以给usb接口变量赋值
        usb.Start()
        usb.Stop()
    }
    func main() {
        //分别创建结构体对象
        c := Computer{}
        p := Phone{"苹果手机", 6999}
        i := IPad{"华为平板", 7999} 
        //
        //手机连接笔记本，插上手机
        c.Working(p)
        fmt.Printf("名字:%v 价格:%d\n", p.Name, p.Price)
        fmt.Println("------------------")
        //平板连接笔记本，插上平板
        c.Working(i)
        fmt.Printf("名字：%v 价格：%d\n", i.Name, i.Price)
    }
```
示例2:
```go
    package main
    import "fmt"
    //员工接口
    type Employer interface {
        CalcSalary() float32
    }
    //开发者
    type Programer struct {
        name  string
        base  float32
        extra float32
    }
    //创建开发者实例
    func NewProgramer(name string, base float32, extra float32) Programer {
        return Programer{
            name,
            base,
            extra,
        }
    }
    //计算开发者工资，实现了CalcSalary方法
    func (p Programer) CalcSalary() float32 {
        return p.base
    }
    //销售群体
    type Sale struct {
        name  string
        base  float32
        extra float32
    }
    //创建销售实例
    func NewSale(name string, base float32, extra float32) Sale {
        return Sale{
            name,
            base,
            extra,
        }
    }
    //实现了CalcSalary方法
    func (p Sale) CalcSalary() float32 {
        return p.base + p.extra*p.base*0.5
    }
    //计算所有人的工资接收参数，接口切片
    func calcAll(e []Employer) float32 {
        /*
            fmt.Println(e)
            [{码云 50000 0} {刘抢东 40000 0} {麻花藤 30000 0} {格格 3000 2.5} {小雪 1800 2.5} {小雨 2000 2.5}]
        */
        var cost float32
        //忽略索引，v是每一个结构体
        for _, v := range e {
            cost = cost + v.CalcSalary()
        }
        return cost
    }
    func main() {
        p1 := NewProgramer("码云", 50000.0, 0)
        p2 := NewProgramer("刘抢东", 40000, 0)
        p3 := NewProgramer("麻花藤", 30000, 0)
        s1 := NewSale("格格", 3000, 2.5)
        s2 := NewSale("小雪", 1800, 2.5)
        s3 := NewSale("小雨", 2000, 2.5)
        var employList []Employer
        employList = append(employList, p1)
        employList = append(employList, p2)
        employList = append(employList, p3)
        employList = append(employList, s1)
        employList = append(employList, s2)
        employList = append(employList, s3)
        cost := calcAll(employList)
        fmt.Printf("这个月人力成本：%f\n", cost)
    }
```

### Go接口细节

    1.接口本身不能创建实例，但是可以指向一个实现了该接口的变量实例，如结构体
    2.接口中所有方法都没有方法体，是没有实现的方法
    3.Go中不仅是struct可以实现接口，自定义类型也可以实现接口，如type myInt int 自定义类型
    4.一个自定义类型，只有实现了某个接口，才可以将自定义类型的实例变量，赋给接口类型，否则报错missing xx method
    5.一个自定义类型，可以实现多个接口(实现多个接口的所有方法)
    6.接口类型不得写入任何变量 如
    type Usb interface{
        method1()
        method2()
        Name string  //错误，编译器不通过
    }
    7.接口A可以继承多个别的接口B、接口C，想要实现A，也必须实现B、C所有方法，称作接口组合
    8.interface类型，默认是指针(引用类型)，如果没初始直接使用，输出nil，可以赋给实现了接口的变量
    9.空接口interface{}，没有任何类型，也就是实现了任何类型，可以吧任何一个变量赋给空接口
    10.匿名组合的接口，不可以有同名方法，否则报错duplicate method

### 参考
```html
https://pythonav.com/wiki/detail/4/54/
```