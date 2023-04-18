- 在一个函数调用前加上“go”关键词，那么本地调用就会在一个新的goroutine 中并发执行

  - 如果该函数有返回值，则会被抛弃
  - 函数结束，goroutine 也会结束

- 不要通过共享内存来通信，而应该通过通信来共享内存

- go通过消息机制而非共享内存作为并发单元间的通信方式，这个消息机制在go中被称为channel

- 消息机制认为每一个并发单元都是一个自包含、独立的个体，并且有自己不共享的变量。

- 每个并发单元的输入输出都只能是消息。

- channel是go在语言级别为goroutine 间的通信方式

- channel是类型相关的，一个channel只能传递一种类型的值

- channel语法

  - 声明

    - `var a chan int`：声明一个传递类型为int的channel
    - `var b map[string] chan bool` ：声明了一个map，元素是bool类型的channel

  - 初始化

    - `a := make(chan int)`：声明并初始化一个int类型名为a的channel 

  - 读取与写入

    - `a <- 1` ：将1写入名为a的channel
    - `value := <- a `：从名为a的channel中读取数据到value中

  - 设置限制大小带有缓冲的channel

    - 在需要传输大量数据的场景下，传递单个数据的channel就不合适啦
    - `ch := make(chan init ,1024)`：声明并创建一个大小1024的int类型的channel
    - 在没有读取方的时候，写入方可以一直写，直到填充完channel前都不会阻塞

- 单向只读只写channel

  - 默认情况下，通道 channel 是双向的，也就是，既可以往里面发送数据也可以同里面接收数据。但是，我们经常见一个通道作为参数进行传递而只希望对方是单向使用的，要么只让它发送数据，要么只让它接收数据，这时候我们可以指定通道的方向。而所谓的单向channel，可以理解为对channel的限制。例如限制某个函数只能往某个channel中写入数据

  -  如果直接使用make创建单向channel（`ch := make(<-chan int)`)，就毫无意义。通常声明初始化一个正常双向的channel，再使用类型转换创建单向的

    ```go
    ch := make(chan int)
    // 声明一个只能写入数据的通道类型, 并赋值为ch
    var chSendOnly chan<- int = ch
    //声明一个只能读取数据的通道类型, 并赋值为ch
    var chRecvOnly <-chan int = ch
    ```

    ```go
    func producer(out chan<- int)  {
       for i:= 0; i < 10; i++ {
          out <- i*i                // 将 i*i 结果写入到只写channel
       }
       close(out)
    }
    // 此通道只能读，不能写
    func consumer(in <-chan int)  {
       for num := range in {        // 从只读channel中获取数据
          fmt.Println("num =", num)
       }
    }
    func main()  {
       ch := make(chan int)     // 创建一个双向channel
       // 新建一个groutine， 模拟生产者，产生数据，写入 channel
       go producer(ch)          // channel传参， 传递的是引用。
       // 主go程，模拟消费者，从channel读数据，打印到屏幕
       consumer(ch)             // 与 producer 传递的是同一个 channel
    }
    ```

- 关闭channel

  - 关闭channel直接使用`close()`即可，但是如何确认channel是否已经关闭?可通过在读取时使用多重返回值的方式进行判断

    ```go
    close(ch)
    x , ok := <- ch
    //这个用法与 map 中的按键获取 value 的过程比较类似，只需要看第二个 bool 返回值即可，如果返回值是 false 则表示 ch 已经被关闭。
    ```

  - *不要从接收端关闭channel，也不要关闭有多个并发发送者的channel*。换句话说，如果sender(发送者)只是唯一的sender或者是channel最后一个活跃的sender，那么你应该在sender的goroutine关闭channel，从而通知receiver(s)(接收者们)已经没有值可以读了。维持这条原则将保证永远不会发生向一个已经关闭的channel发送值或者关闭一个已经关闭的channel。

- channel的读写堵塞、超时问题的解决

  - **问题**：如果往channel中写数据，此时发现channel满了；如果从channel中读取数据，此时发现channel是空的，如果此时没有处理逻辑，会造成个goroutine 堵塞锁死

  - **解决**：使用select实现给channel的读写设置超时机制

    ```go
    func main() {
        ch := make(chan int)
        quit := make(chan bool)
        //新开一个协程
        go func() {
            for {
                select {
                  	// select的每个 case 语句里必须是一个 IO 操作
                  	// 如果ch成功读到数据，则进行该case处理语句
                    case num := <-ch:
                    	fmt.Println("num = ", num)
                    case <-time.After(3 * time.Second):
                    	fmt.Println("超时")
                    	quit <- true
                }
            }
        }() //别忘了()
      
        for i := 0; i < 5; i++ {
            ch <- i
            time.Sleep(time.Second)
        }
      
        <-quit
        fmt.Println("程序结束")
    }
    ```

