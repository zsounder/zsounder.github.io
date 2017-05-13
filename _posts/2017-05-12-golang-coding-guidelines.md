---
layout: post
title:  "Golang Coding Guidelines"
date:   2017-05-13 15:14:54
categories: Golang
tags: Golang
---

* content
{:toc}

## 命名
1. 文件名使用单个英文单词，如 http.go，多个英文单词之间不用 `_` ，如httputil.go。
2. 测试代码文件名使用 _test.go 结尾。
3. 变量名采用驼峰标准
	 - 首字符大小写决定是否导出
	 - 不要使用`_` 来命名变量名，多个变量申明放在一起。
4. 接口名统一使用IF结尾。
5. 不要使用反逻辑来命名，IsEnabled，IsNotEnabled。
6. package的名字与目录一致，包名为小写单词，不使用下划线或者混合大小写。
7. 可以使用缩写，但需要有一些注释，使用较通用的缩写。

## import
1. 不要使用相对路径引入包
2. 有顺序的引入包：不同的类型采用空格分离，实标准库 -> 项目包 -> 第三方包。
    ```go
    import (
        "errors"
        "fmt"
        "strconv"
        "strings"
        "time"

        "github.com/to-your-path/golib"

        "github.com/pquerna/ffjson/ffjson"
    )
    ```

## 注释
1. 包注释：每个程序包都应该有一个包注释，位于package子句之前的块注释或行注释，注释内容和包之间不能有空行，单行注释使用//，多行注释使用/* */包裹，独立doc.go完成包注释编写。
2. 可导出类型：第一条语句应该为一条概括语句，并且使用被声明的名字作为开头
3. 注释应该是一个完整的句子，末尾以.或者。结束
    ```go
    /*
    Package regexp implements a simple library for regular expressions.

    The syntax of the regular expressions accepted is:

        regexp:
            concatenation { '|' concatenation }
        concatenation:
            { closure }
        closure:
            term [ '*' | '+' | '?' ]
        term:
            '^'
            '$'
            '.'
            character
            '[' [ '^' ] character-ranges ']'
            '(' regexp ')'
    */
    package regexp

    // Fprint formats using the default formats for its operands and writes to w.
    // Spaces are added between operands when neither is a string.
    // It returns the number of bytes written and any write error encountered.
    func Fprint(w io.Writer, a ...interface{}) (n int, error os.Error) {
    }
    ```

## 代码风格、健壮性检查
1. 使用go fmt格式化代码。
2. 使用tab进行缩进，这是gofmt的缺省输出。只有在你必须的时候才使用空格。
3. 行长度，100字符，感觉一行太长，可以折成几行，并额外使用一个tab进行缩进。
4. [errcheck](https://github.com/kisielk/errcheck), [golint](https://github.com/bytbox/golint)

## interface
1. 当函数返回interface时，显式返回nil。

    `错误方式`
    ```go
    package main

    import "fmt"

    func main() {
      doit := func(arg int) interface{} {
        var result *struct{} = nil
        if arg > 0 {
          result = &struct{}{}
        }
        return result
      }
      if res := doit(-1); res != nil {
        fmt.Println("good result:", res)
        // prints: good result: <nil>
        //'res' is not 'nil', but its value is 'nil'
      }
    }
    ```
    `正确方式`
    ```go
    package main

    import "fmt"

    func main() {
      doit := func(arg int) interface{} {
        var result *struct{} = nil
        if arg > 0 {
          result = &struct{}{}
        } else {
          return nil //return an explicit 'nil'
        }
        return result
      }

      if res := doit(-1); res != nil {
        fmt.Println("good result:", res)
      } else {
        fmt.Println("bad result (res is nil)")
        //here as expected
      }
    }
    ```
    >interface作为两个成员实现：一个类型和一个值

## Struct相关
1. 初始化Struct使用标签语法，否则添加一个新的字段到T结构体，代码会编译失败。
2. 将Struct的初始化拆分到多行
3. 禁止使用 me、this、self作为Receiver Name，保持同一个struct中的Receiver Name命名一致，建议统一采用字母p命名。
4. Receiver使用值或指针类型，依据数据是否被改变的原则
	- 不清楚是否指针或值，统一采用指针型；
    - 接收者是map、slice、channel，不要用指针传递，如果需要对slice进行修改，通过返回值的方式重新赋值；
    - 接收者是含有sync.Mutex或者类似同步字段的结构体，必须使用指针传递避免复制；
    - 接收者是大的结构体或者数组，使用指针传递会更有效率。
5. Get/Set方法，如果你有一个域叫做owner（小写，不被导出），则Get方法应该叫做Owner（大写，被导出），Set方法，可以叫做SetOwner。
    ```go
    //不要采用
    T{Foo: "example", Bar:someLongVariable, Qux:anotherLongVariable, B: forgetToAddThisToo}

    //采用
    T{
        Foo: "example",
        Bar: someLongVariable,
        Qux: anotherLongVariable,
        B: forgetToAddThisToo,
    }

    func(p  TestStruct) Tally(something)int    // p不会有任何改变
    func(p *TestStruct)Tally(something)int    // p会改变数据
    ```

## 错误处理
1. 不丢弃任何有返回err的调用，不要采用`_`丢弃，必须全部处理(log)。
2. 错误描述如果是英文必须为小写，不需要标点结尾。
3. 在逻辑处理中禁用panic，package对外的接口不能有panic，只能在包内采用。
4. 采用独立的错误流进行处理。
    ```go
    // 不要采用这种方式
    if err != nil {
         // error handling
    } else {
         // normal code
    }

    //采用下面的方式
    if err != nil {
         // error handling
         return // or continue, etc.
    }
    // normal code

    //如果返回值需要初始化，则采用下面的方式,if中声明的变量作用域同if语句
    x, err := f()
    if err != nil {
         // error handling
         return
    }
    // use x
    ```

## 控制语句
1.  不添加多余的()，遍历数组、切片、字符串、map、channel中数据优先range
2.  没有逗号操作符，++\--是语句(非表达式)，for中注意并行赋值
    ```go
    for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
        a[i], a[j] = a[j], a[i]
    }
    ```
3.  switch case是按照从上到下顺序进行求值，直到匹配，不会自动从一个case子句跌落到下一个case子句，多个匹配条件时使用`,`分隔。
    ```go
    func shouldEscape(c byte) bool {
        switch c {
        case ' ', '?', '&', '=', '#', '+', '%':
            return true
        }
        return false
    }
    ```
4.  range，如果只需要第一个值，则丢弃第二个，如果只需要第二个，则第一个置为`_`。
    ```go
    for key, value := range Servers {
        // something using key and value
    }
    for _, value := range Servers {
        // something using value
    }
    for key := range Servers {
        // something using key
    }
    ```
5.  if接受初始化语句，约定如下方式建立局部变量
    ```go
      if err := file.Chmod(0664); err != nil {
         return err
    }
    ```
6. select语句，若干个 case 都不 block，随机选一个执行
    ```go
    func (this *PipeService) Run(closeSig chan bool) {
      for {
        select {
        case <-closeSig:
          return
        case call := <-this.TaskPipe.Queue:
          this.TaskPipe.RunRoutineCall(call)
        case call := <-this.TimeCall.Queue:
          call.CallBack()
        }
      }
    }
    ```

## 函数
1. 不要结巴，~~bytes.ByteBuffer~~ `bytes.Buffer`。
2. 函数采用命名的多值返回，便于理解，传入变量和返回变量以小写字母开头。
3. 参数类型
	- 对于少量数据，不要传递指针，整数，浮点和复数，字符串, 小结构体等
	- 对于大量数据的struct可以考虑使用指针
	- map，slice，chan引用类型，传入时不使用指针
	- 包含 sysnc.Mutex 的 struct ，则需要使用指针，避免对象复制。
4. defer函数善后
	- 后进先出
	- 执行return -> 写入返回值  -> 执行defer -> 函数携带返回值退出
	- defer参数的求值在defer声明时(非执行时)
    ```go
    package main

    import "fmt"

    func main() {
      var i int = 1
      defer fmt.Println("result =>", func() int { return i * 2 }())
      i++
      //prints: result => 2 (not ok if you expected 4)
    }
    ```

## init函数
1.  init函数只完成以下功能
	 - 屏蔽底层平台差异
	 - 包初始化(全局变量等)
	 - 逻辑无关
2.  建议一个package中只有一个init函数

## 内存
1. 内建函数new(T)，分配内存+置零值，返回类型为*T的值。
2. 内建函数make(T, args)，分配内存+初始化，创建slice\map\channel，返回类型为T的值。
3. 创建空slice，避免在正式使用前分配内存。
    ```go
    var t []string  //优先使用
    t := []string{}
    ```
4. 优先传递slice，数组是值，将一个数组赋值给另一个，会拷贝所有的元素，性能问题。
5. 如果map的value较大，使用指针来存储，避免性能问题，类似的还有channel，slice等
6. 字符串不是以`\x0`结束作为判断的， len()函数返回byte的数量
7. list删除`nil`元素会crash
8. 慎重在slice上划分新的slice，导致难以预期的内存使用，考虑拷贝
9. 变量存储于堆或栈由编译器根据变量大小和泄露分析的结果决定，可返回局部变量的指针。
10. 在函数外部申明必须使用 `var`,不要采用 `:=`，容易踩到变量的作用域的问题。
11. `:=`定义的变量，注意由于作用域不同引起的遮盖问题。
12. 函数外部申明必须使用var, 不要采用:=，容易踩到变量的作用域的问题
    ```go
    //testpointer.go
    package main
    import "fmt"

    var p *int
    func foo() (*int, error) {
            var i int = 5
            return &i, nil
    }
    func bar() {
            //use p
            fmt.Println(*p)
            // panic: runtime error:
            // invalid memory address or nil pointer dereference
    }
    func main() {
            p, err := foo()
            if err != nil {
                    fmt.Println(err)
                    return
            }
            bar()
            fmt.Println(*p)
    }
    ```
12. 内存对齐，建议占用内存高->底排序，[Issue10014](https://github.com/golang/go/issues/10014)
    ```go
    struct{ bool; float64; int16 }
    struct{ bool; int16; float64 }
    ```
    ```go
	struct{ float64; int16; bool }
    ```

## Map
1. 不依赖map中的元素顺序，range时是无序的。
2. 建议封装Map的Get/Set操作，Map非goroutine安全。
3. 删除元素时无需检测是否存在，key不在map也可删除
5. 使用struct{}作为标记值，而不是bool或者interface{}，节省内存
    > The struct type with no fields is called the empty struct, written struct{}. It has size zero and carries no information but may be useful nonetheless. Some Go programmers use it instead of bool as the value type of a map that represents a set, to emphasize that only the keys are significant, but the space saving is marginal and the syntax more cumbersome, so we generally avoid it.Go lets us declare a field with a type but no name; such fields are called anonymous fields. The type of the field must be a named type or a pointer to a named type. —— [The Go Programming Language](http://www.gopl.io/)

    ```go
    // UniqStr returns a copy if the passed slice with only unique string results.
    func UniqStr(col []string) []string {
      m := map[string]struct{}{}
      for _, v := range col {
        if _, ok := m[v]; !ok {
          m[v] = struct{}{}
        }
      }
      list := make([]string, len(m))

      i := 0
      for v := range m {
        list[i] = v
        i++
      }
      return list
    }
    ```

## 接口检查
如接口的类型转换和检查无法再编译阶段静态完成，需确保某个类型确实满足某个接口的定义,则用`_`做一个全局声明(强制编译期完成检查)
```go
var _ json.Marshaler = (*RawMessage)(nil)
```

## channel
1. 谁创建的通道，谁负责关闭。
	- 发送端关闭通道后，接收端仍然可以正常接收通道中已有的数据；
	- 接收端不可关闭channel，无法判断发送端是否还有数据要发送。
2. 保证channel初始化后再进行写入、读取，否则会造成当前goroutine堵塞。
3. 向关闭的channel发送值，会引发panic。
4. 关闭已关闭的channel, 会引发panic。
5. 被channel传递的是值的副本,而不是值本身。

## 并发与goroutine
1. cheap not free，测试go 1.5, 每个goroutine初始占用 3K的内存，创建2us左右，预见并控制goroutine的总数
2. 避免滥用goroutine，不合理的并行会带来更多的bug，也让问题难以复现和调
3. 逻辑中不出现并发支持
    ```go
    func doConcurrently(job string, err chan error) {
        go func() {
          // do something
            err <- errors.New("something went wrong!")
        }()
    }

    func main() {
        jobs := []string{"one", "two", "three"}
        errc := make(chan error)
        for _, job := range jobs {
            doConcurrently(job, errc)
        }
        // something else
    }
    ```
    ```go
    func do(job string) error {
        // do something
        return errors.New("something went wrong!")
    }
    func main() {
        jobs := []string{"one", "two", "three"}
        errc := make(chan error)
        for _, job := range jobs {
            go func(job string) {
                errc <- do(job)
            }(job)
        }
        // something else
    }
    ```

## 程序监控
随时观察程序状态(goroutine, heap, thread)，使用[pprof](https://blog.golang.org/profiling-go-programs)工具。
```go
import _ "net/http/pprof“
go func() {
    http.ListenAndServe("localhost:6060", nil)
}()
```

```shell
go tool pprof http://localhost:6060/debug/pprof
```
