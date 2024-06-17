---
title: Go 的并发
date: 2024-04-28 11:14:27
abstract: Go 语言的并发机制简介
tags:
    - Go
    - Concurrency
    - Goroutine
    - Channel
categories:
    - Blog
---

# Goroutines 和 channels

## 8.1 Goroutines

在 Go 语言中，每一个并发的执行单元叫作一个 goroutine。可以简单地把 goroutine 类比作一个线程，这样就可以写出一些正确的程序了。goroutine 和线程的本质区别会在 9.8 节中说明。

当一个程序启动时，其主函数即在一个单独的 goroutine 中运行，我们叫它 main goroutine。新的 goroutine 会用 go 语句来创建。在语法上，go 语句是一个普通的函数或方法调用前加上关键字 go。go 语句会使其语句中的函数在一个新创建的 goroutine 中运行。而 go 语句本身会迅速地完成。

```go
f()    // call f(); wait for it to return
go f() // create a new goroutine that calls f(); don't wait
```

示例：

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

动画显示了几秒以后，fib(45) 的调用成功地返回，并打印结果：

```text
Fibonacci(45) = 1134903170
```

然后主函数返回。主函数返回时，所有的 goroutine 都会被直接打断，程序退出。

除了从主函数退出或者直接终止程序之外，没有其它的编程方法能够让一个 goroutine 来打断另一个的执行，但是之后可以看到一种方式来实现这个目的，通过 goroutine 之间的通信来让一个 goroutine 请求其它的 goroutine，并让被请求的 goroutine 自行结束执行。

## 8.2 示例：并发的 Clock 服务

```go
// Netcat1 is a read-only TCP client.
package main

import (
    "io"
    "log"
    "net"
    "os"
)

func main() {
    conn, err := net.Dial("tcp", "localhost:8000")
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()
    mustCopy(os.Stdout, conn) // 此时服务器端同时只能处理一个客户端连接，因为这个处理函数是阻塞执行的
}

func mustCopy(dst io.Writer, src io.Reader) {
    if _, err := io.Copy(dst, src); err != nil {
        log.Fatal(err)
    }
}
```

```go
for {
    conn, err := listener.Accept()
    if err != nil {
        log.Print(err) // e.g., connection aborted
        continue
    }
    go handleConn(conn) // handle connections concurrently
}
```

将处理函数改为 go 执行，确保其并发能力。

## 8.3 示例：并发的 Echo 服务

在使用 go 关键词的同时，需要慎重地考虑 net.Conn 中的方法在并发地调用时是否安全，事实上对于大多数类型来说也确实不安全。

## 8.4 channels

如果说 goroutine 是 Go 语言程序的并发体的话，那么 channels 则是它们之间的通信机制。一个 channel 是一个通信机制，它可以让一个 goroutine 通过它给另一个 goroutine 发送值信息。每个 channel 都有一个特殊的类型，也就是 channels 可发送数据的类型。一个可以发送 int 类型数据的 channel 一般写为 `chan int`。

使用内置的 make 函数，我们可以创建一个 channel：

```go
ch := make(chan int) // ch has type 'chan int'
```

和 map 类似，channel 也对应一个 make 创建的底层数据结构的引用。当我们复制一个 channel 或用于函数参数传递时，我们只是拷贝了一个 channel 引用，因此调用者和被调用者将引用同一个 channel 对象。和其它的引用类型一样，channel 的零值也是 nil。

两个相同类型的 channel 可以使用 == 运算符比较。如果两个 channel 引用的是相同的对象，那么比较的结果为真。一个 channel 也可以和 nil 进行比较。

一个 channel 有发送和接受两个主要操作，都是通信行为。一个发送语句将一个值从一个 goroutine 通过 channel 发送到另一个执行接收操作的 goroutine。发送和接收两个操作都使用 `<-` 运算符。在发送语句中，`<-` 运算符分割 channel 和要发送的值。在接收语句中，`<-` 运算符写在 channel 对象之前。一个不使用接收结果的接收操作也是合法的。

```go
ch <- x  // a send statement
x = <-ch // a receive expression in an assignment statement
<-ch     // a receive statement; result is discarded
```

Channel 还支持 close 操作，用于关闭 channel，随后对基于该 channel 的任何发送操作都将导致 panic 异常。对一个已经被 close 过的 channel 进行接收操作依然可以接受到之前已经成功发送的数据；如果 channel 中已经没有数据的话将产生一个零值的数据。

使用内置的 close 函数就可以关闭一个 channel：

```go
close(ch)
```

### 8.4.1 不带缓存的 Channels

一个基于无缓存 Channels 的发送操作将导致发送者 goroutine 阻塞，直到另一个 goroutine 在相同的 Channels 上执行接收操作，当发送的值通过 Channels 成功传输之后，两个 goroutine 可以继续执行后面的语句。反之，如果接收操作先发生，那么接收者 goroutine 也将阻塞，直到有另一个 goroutine 在相同的 Channels 上执行发送操作。

基于无缓存 Channels 的发送和接收操作将导致两个 goroutine 做一次同步操作。因为这个原因，无缓存 Channels 有时候也被称为同步 Channels。当通过一个无缓存 Channels 发送数据时，接收者收到数据发生在再次唤醒发送者 goroutine 之前（happens before）。

在讨论并发编程时，当我们说 x 事件在 y 事件之前发生（happens before），我们并不是说 x 事件在时间上比 y 时间更早；我们要表达的意思是要保证在此之前的事件都已经完成了，例如在此之前的更新某些变量的操作已经完成，你可以放心依赖这些已完成的事件了。

当我们说 x 件既不是在 y 事件之前发生也不是在 y 事件之后发生，我们就说 x 事件和 y 事件是并发的。这并不是意味着 x 事件和 y 事件就一定是同时发生的，我们只是不能确定这两个事件发生的先后顺序。在下一章中我们将看到，当两个 goroutine 并发访问了相同的变量时，我们有必要保证某些事件的执行顺序，以避免出现某些并发问题。

基于 channels 发送消息有两个重要方面。首先每个消息都有一个值，但是有时候通讯的事实和发生的时刻也同样重要。当我们更希望强调通讯发生的时刻时，我们将它称为消息事件。有些消息事件并不携带额外的信息，它仅仅是用作两个 goroutine 之间的同步，这时候我们可以用 struct{} 空结构体作为 channels 元素的类型，虽然也可以使用 bool 或 int 类型实现同样的功能，`done <- 1` 语句也比 `done <- struct{}{}` 更短。

### 8.4.2 串联的 Channels（Pipeline）

如果发送者知道，没有更多的值需要发送到 channel 的话，那么让接收者也能及时知道没有多余的值可接收将是有用的，因为接收者可以停止不必要的接收等待。这可以通过内置的 close 函数来关闭 channel 实现：

```go
close(naturals)
```

当一个 channel 被关闭后，再向该 channel 发送数据将导致 panic 异常。当一个被关闭的 channel 中已经发送的数据都被成功接收后，后续的接收操作将不再阻塞，它们会立即返回一个零值。

有办法直接测试一个 channel 是否被关闭，但是接收操作有一个变体形式：它多接收一个结果，多接收的第二个结果是一个布尔值 ok，ture 表示成功从 channels 接收到值，false 表示 channels 已经被关闭并且里面没有值可接收。使用这个特性，我们可以修改 squarer 函数中的循环代码，当 naturals 对应的 channel 被关闭并没有值可接收时跳出循环，并且也关闭 squares 对应的 channel。

```go
// Squarer
go func() {
    for {
        x, ok := <-naturals
        if !ok {
            break // channel was closed and drained
        }
        squares <- x * x
    }
    close(squares)
}()
```

因为上面的语法是笨拙的，而且这种处理模式很常见，因此 Go 语言的 range 循环可直接在 channels 上面迭代。使用 range 循环是上面处理模式的简洁语法，它依次从 channel 接收数据，当 channel 被关闭并且没有值可接收时跳出循环。

在下面的改进中，我们的计数器 goroutine 只生成 100 个含数字的序列，然后关闭 naturals 对应的 channel，这将导致计算平方数的 squarer 对应的 goroutine 可以正常终止循环并关闭 squares 对应的 channel。（在一个更复杂的程序中，可以通过 defer 语句关闭对应的 channel。）最后，主 goroutine 也可以正常终止循环并退出程序。

```go
func main() {
    naturals := make(chan int)
    squares := make(chan int)

    // Counter
    go func() {
        for x := 0; x < 100; x++ {
            naturals <- x
        }
        close(naturals)
    }()

    // Squarer
    go func() {
        for x := range naturals {
            squares <- x * x
        }
        close(squares)
    }()

    // Printer (in main goroutine)
    for x := range squares {
        fmt.Println(x)
    }
}
```

其实你并不需要关闭每一个 channel。只有当需要告诉接收者 goroutine，所有的数据已经全部发送时才需要关闭 channel。不管一个 channel 是否被关闭，当它没有被引用时将会被 Go 语言的垃圾自动回收器回收。（不要将关闭一个打开文件的操作和关闭一个 channel 操作混淆。对于每个打开的文件，都需要在不使用的时候调用对应的 Close 方法来关闭文件。）

试图重复关闭一个 channel 将导致 panic 异常，试图关闭一个 nil 值的 channel 也将导致 panic 异常。关闭一个 channels 还会触发一个广播机制。

### 8.4.3 单方向的 Channel

当一个 channel 作为一个函数参数时，它一般总是被专门用于只发送或者只接收。

为了表明这种意图并防止被滥用，Go 语言的类型系统提供了单方向的 channel 类型，分别用于只发送或只接收的 channel。类型 `chan<- int` 表示一个只发送 int 的 channel，只能发送不能接收。相反，类型 `<-chan int` 表示一个只接收 int 的 channel，只能接收不能发送。（箭头 `<-` 和关键字 chan 的相对位置表明了 channel 的方向。）这种限制将在编译期检测。

因为关闭操作只用于断言不再向 channel 发送新的数据，所以只有在发送者所在的 goroutine 才会调用 close 函数，因此对一个只接收的 channel 调用 close 将是一个编译错误。

这是改进的版本，这一次参数使用了单方向 channel 类型：

```go
func counter(out chan<- int) {
    for x := 0; x < 100; x++ {
        out <- x
    }
    close(out)
}

func squarer(out chan<- int, in <-chan int) {
    for v := range in {
        out <- v * v
    }
    close(out)
}

func printer(in <-chan int) {
    for v := range in {
        fmt.Println(v)
    }
}

func main() {
    naturals := make(chan int)
    squares := make(chan int)
    go counter(naturals)
    go squarer(squares, naturals)
    printer(squares)
}
```

调用 `counter(naturals)` 时，naturals 的类型将隐式地从 chan int 转换成 `chan<- int`。调用 `printer(squares)` 也会导致相似的隐式转换，这一次是转换为 `<-chan int` 类型只接收型的 channel。任何双向 channel 向单向 channel 变量的赋值操作都将导致该隐式转换。这里并没有反向转换的语法：也就是不能将一个类似 `chan<- int` 类型的单向型的 channel 转换为 `chan int` 类型的双向型的 channel。

### 8.4.4 带缓存的 channels

带缓存的 Channel 内部持有一个元素队列。队列的最大容量是在调用 make 函数创建 channel 时通过第二个参数指定的。下面的语句创建了一个可以持有三个字符串元素的带缓存 Channel。

```go
ch = make(chan string, 3)
```

向缓存 Channel 的发送操作就是向内部缓存队列的尾部插入元素，接收操作则是从队列的头部删除元素。如果内部缓存队列是满的，那么发送操作将阻塞直到因另一个 goroutine 执行接收操作而释放了新的队列空间。相反，如果 channel 是空的，接收操作将阻塞直到有另一个 goroutine 执行发送操作而向队列插入元素。

在某些特殊情况下，程序可能需要知道 channel 内部缓存的容量，可以用内置的 cap 函数获取：

```go
fmt.Println(cap(ch)) // "3"
```

同样，对于内置的 len 函数，如果传入的是 channel，那么将返回 channel 内部缓存队列中有效元素的个数。因为在并发程序中该信息会随着接收操作而失效，但是它对某些故障诊断和性能优化会有帮助。

```go
fmt.Println(len(ch)) // "2"
```

Go 语言新手有时候会将一个带缓存的 channel 当作同一个 goroutine 中的队列使用，虽然语法看似简单，但实际上这是一个错误。Channel 和 goroutine 的调度器机制是紧密相连的，如果没有其他 goroutine 从 channel 接收，发送者——或许是整个程序——将会面临永远阻塞的风险。如果你只是需要一个简单的队列，使用 slice 就可以了。

下面的例子展示了一个使用了带缓存 channel 的应用。它并发地向三个镜像站点发出请求，三个镜像站点分散在不同的地理位置。它们分别将收到的响应发送到带缓存 channel，最后接收者只接收第一个收到的响应，也就是最快的那个响应。因此 mirroredQuery 函数可能在另外两个响应慢的镜像站点响应之前就返回了结果。（顺便说一下，多个 goroutines 并发地向同一个 channel 发送数据，或从同一个 channel 接收数据都是常见的用法。）

```go
func mirroredQuery() string {
    responses := make(chan string, 3)
    go func() { responses <- request("asia.gopl.io") }()
    go func() { responses <- request("europe.gopl.io") }()
    go func() { responses <- request("americas.gopl.io") }()
    return <-responses // return the quickest response
}

func request(hostname string) (response string) { /* ... */ }
```

如果我们使用了无缓存的 channel，那么两个慢的 goroutines 将会因为没有人接收而被永远卡住。这种情况，称为 goroutines 泄漏，这将是一个 BUG。和垃圾变量不同，泄漏的 goroutines 并不会被自动回收，因此确保每个不再需要的 goroutine 能正常退出是重要的。

## 8.5 并发的循环

```go
// makeThumbnails3 makes thumbnails of the specified files in parallel.
func makeThumbnails3(filenames []string) {
    ch := make(chan struct{})
    for _, f := range filenames {
        go func(f string) {
            thumbnail.ImageFile(f) // NOTE: ignoring errors
            ch <- struct{}{}
        }(f)
    }
    // Wait for goroutines to complete.
    for range filenames {
        <-ch
    }
}
```

ch 用于控制循环中所有的文件处理完成后才令函数返回。而且，goroutine 执行的方式是 `go func(f string)`，而不是直接使用闭包的形式：

```go
for _, f := range filenames {
    go func() {
        thumbnail.ImageFile(f) // NOTE: incorrect!
        // ...
    }()
}
```

回顾 5.6.1 章节内容，这样可以确保单独变量 f 不会被所有的匿名函数值所共享。显式地添加这个参数，我们能够确保使用的 f 是当 go 语句执行时的“当前”那个 f。

```go
// makeThumbnails5 makes thumbnails for the specified files in parallel.
// It returns the generated file names in an arbitrary order,
// or an error if any step failed.
func makeThumbnails5(filenames []string) (thumbfiles []string, err error) {
    type item struct {
        thumbfile string
        err       error
    }

    ch := make(chan item, len(filenames))
    for _, f := range filenames {
        go func(f string) {
            var it item
            it.thumbfile, it.err = thumbnail.ImageFile(f)
            ch <- it
        }(f)
    }

    for range filenames {
        it := <-ch
        if it.err != nil {
            return nil, it.err
        }
        thumbfiles = append(thumbfiles, it.thumbfile)
    }

    return thumbfiles, nil
}
```

## 8.6 示例：并发的 Web 爬虫

## 8.7 基于 select 的多路复用

```go
select {
case <-ch1:
    // ...
case x := <-ch2:
    // ...use x...
case ch3 <- y:
    // ...
default:
    // ...
}
```

select 会等待 case 中有能够执行的 case 时去执行。当条件满足时，select 才会去通信并执行 case 之后的语句；这时候其它通信是不会执行的。一个没有任何 case 的 select 语句写作 select{}，会永远地等待下去。

有时候我们希望能够从 channel 中发送或者接收值，并避免因为发送或者接收导致的阻塞，尤其是当 channel 没有准备好写或者读时。select 语句就可以实现这样的功能。select 会有一个 default 来设置当其它的操作都不能够马上被处理时程序需要执行哪些逻辑。

下面的 select 语句会在 abort channel 中有值时，从其中接收值；无值时什么都不做。这是一个非阻塞的接收操作；反复地做这样的操作叫做“轮询 channel”。

```go
select {
case <-abort:
    fmt.Printf("Launch aborted!\n")
    return
default:
    // do nothing
}
```

## 8.8 示例：并发的目录遍历

```go
// walkDir recursively walks the file tree rooted at dir
// and sends the size of each found file on fileSizes.
func walkDir(dir string, fileSizes chan<- int64) {
    for _, entry := range dirents(dir) {
        if entry.IsDir() {
            subdir := filepath.Join(dir, entry.Name())
            walkDir(subdir, fileSizes)
        } else {
            fileSizes <- entry.Size()
        }
    }
}

// dirents returns the entries of directory dir.
func dirents(dir string) []os.FileInfo {
    entries, err := ioutil.ReadDir(dir)
    if err != nil {
        fmt.Fprintf(os.Stderr, "du1: %v\n", err)
        return nil
    }
    return entries
}
```

```go
func main() {
    // ...determine roots...
    // Traverse each root of the file tree in parallel.
    fileSizes := make(chan int64)
    var n sync.WaitGroup
    for _, root := range roots {
        n.Add(1)
        go walkDir(root, &n, fileSizes)
    }
    go func() {
        n.Wait()
        close(fileSizes)
    }()
    // ...select loop...
}

func walkDir(dir string, n *sync.WaitGroup, fileSizes chan<- int64) {
    defer n.Done()
    for _, entry := range dirents(dir) {
        if entry.IsDir() {
            n.Add(1)
            subdir := filepath.Join(dir, entry.Name())
            go walkDir(subdir, n, fileSizes)
        } else {
            fileSizes <- entry.Size()
        }
    }
}
```

由于这个程序在高峰期会创建成百上千的 goroutine，我们需要修改 dirents 函数，用计数信号量来阻止他同时打开太多的文件，就像我们在 8.7 节中的并发爬虫一样：

```go
// sema is a counting semaphore for limiting concurrency in dirents.
var sema = make(chan struct{}, 20)

// dirents returns the entries of directory dir.
func dirents(dir string) []os.FileInfo {
    sema <- struct{}{}        // acquire token
    defer func() { <-sema }() // release token
    // ...
```

## 8.9 并发的退出

```go
var done = make(chan struct{})

func cancelled() bool {
    select {
    case <-done:
        return true
    default:
        return false
    }
}
```

## 8.10 示例：聊天服务

```go
func main() {
    listener, err := net.Listen("tcp", "localhost:8000")
    if err != nil {
        log.Fatal(err)
    }
    go broadcaster()
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Print(err)
            continue
        }
        go handleConn(conn)
    }
}
```

```go
type client chan<- string // an outgoing message channel

var (
    entering = make(chan client)
    leaving  = make(chan client)
    messages = make(chan string) // all incoming client messages
)

func broadcaster() {
    clients := make(map[client]bool) // all connected clients
    for {
        select {
        case msg := <-messages:
            // Broadcast incoming message to all
            // clients' outgoing message channels.
            for cli := range clients {
                cli <- msg
            }
        case cli := <-entering:
            clients[cli] = true

        case cli := <-leaving:
            delete(clients, cli)
            close(cli)
        }
    }
}
```

```go
func handleConn(conn net.Conn) {
    ch := make(chan string) // outgoing client messages
    go clientWriter(conn, ch)

    who := conn.RemoteAddr().String()
    ch <- "You are " + who
    messages <- who + " has arrived"
    entering <- ch

    input := bufio.NewScanner(conn)
    for input.Scan() {
        messages <- who + ": " + input.Text()
    }
    // NOTE: ignoring potential errors from input.Err()

    leaving <- ch
    messages <- who + " has left"
    conn.Close()
}

func clientWriter(conn net.Conn, ch <-chan string) {
    for msg := range ch {
        fmt.Fprintln(conn, msg) // NOTE: ignoring network errors
    }
}
```
