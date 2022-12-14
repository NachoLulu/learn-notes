# Go语言精进之路

> 读书笔记

## 典型项目结构

Go语言典型项目结构按照用途可分为四类

#### 最小标准布局

Go官方并未给出应用项目的标准布局是怎样的，不过Go语言的技术负责人 Russ Cox曾在一个开源项目的issue上给出过他关于最小标准布局的一个想法。

```
-go.mod
-LICENSE
-xx.go
-yy.go
...
或
-go.mod
-LICENSE
-package1
	-package1.go
-package2
	-package2.go
```

pkg、docs、cmd这些目录不应该成为最小标准布局的标准结构的一部分，Go社区崇尚`简单`，故此标准结构足以灵活满足各种Go项目的需求。

#### 以构建二进制可执行文件为用途

关于此用途，其实Go社区也未给出一个标准的布局格式，即为Go社区中`非官方标准的建议结构布局`，也就是Gopher之间们默认约定成俗一种用途结构。

```
GoProject
|-LICENSE
|-Makefile
|-README.md
|-cmd/
|	|-app1/
|	|-main.go
|	|-app2/
|	|-main.go
|-go.mod
|-go.sum
|-pkg/
|	|-lib1/
|	|-lib1.go
|	|-lib2/
|	|-lib2.go
|-[vendor/]
```

pkg目录下主要是存放项目自身实现或引用的一些库文件，该目录下的包可被外部项目引用，算是个项目导出包的一个聚合。并且在Go语言1.4版本中去掉了此层目录，将所有包平铺到项目的根目录下，我认为当项目结构并不复杂的情况下可以此样设计，但是当项目结构或者内嵌的库文件过多时，建议还是使用此层目录，防止显着拥挤，保持简洁。

Makefile只是项目构建工具所用脚本的一个`代表`，可代表任何第三方构建工具所用的脚本，例：Jenkins使用的jenkinsfile；当然当构建脚本过多的时候可以使用build目录存放。

vendor目录为可选目录，按照需求来选择是否使用；vendor是Go1.5版本引入的用于项目本地缓存特定版本以来包的机制，现在Go的包管理工具一般都是使用Go module，当然Go module机制也保留了vendor目录（通过 `go mod vendor`可以生成vendor下的依赖包通过`go build -mod=vendor`可以实现基于vendor目录的构建），因此这里对vendor作为一个可选目录。

#### 以只为构建库为用途

此用途即为只为提供自身的库文件供其他项目调用。

```
GoLibProject
|-LICENSE
|-Makefile
|-README.md
|-go.mod
|-go.sum
|-lib.go
|-lib1/
|	|-lib1.go
|-lib2/
|	|-lib2.go
```

与构建二进制文件用途对比，去掉了cmd、pkg两个子目录，由于构建库没必要存放二进制文件的源码文件，并且由于此类项目就是为了对外开放暴露API接口，所以就没必要存放pkg目录。

vendor也不再作为可选目录了，因为构建库仅通过go.mod进行管理，故venrdor也就没有了存在的必要了。

#### 关于internal目录

无论是上述哪种类型的Go项目，对于不想暴露给外部使用，仅限于项目内部使用的包，在项目结构想可以通过Go1.4版本中引入的internal包机制来实现。

以库文件为例，最简单就是在顶层加个internal目录，将不想暴露到外部的包都存放到该目录下，比如下面的lib1、lib2：

```
GoLibProject
|-LICENSE
|-Makefile
|-README.md
|-go.mod
|-internal/
|	|-lib1/
|	|-lib2/
|-lib.go
|-lib1/
|	|-lib1.go
|-lib2/
|	|-lib2.go
```

根据Go internal机制的作用原理，internal目录下的lib1、lib2可被以GoLibProject作为根目录的其他目录的代码所导入和调用，但是不可被不以GoLibProject作为根目录以外的代码所引用使用。

## 命名惯例

命名保持简单一致性即可。

#### 包

一般建议以小写形式的单个单词命名；无需考虑是否重名，在Go中包名可以不唯一；包名应尽量与包导入路径（import path）的最后一个路径分段保持一致；命名时不仅要考虑包自身名字，还要兼顾该包导出的标识符（变量、常量、类型、函数等）的命名，尽量不再读得时候出现`口吃`。

#### 变量、类型、函数和方法

> 1. 循环和条件变量多采用单个字符命名
> 2. 函数/方法的参数和返回值变量以单个单词或者单个字符为主
> 3. 由于方法在调用时会绑定类型信息，因此命名以单个单词为主
> 4. 函数多以多个单词的复合词进行命名
> 5. 类型多以多个单词的复合词进行命名

多个词组成时局部变量`小驼峰`，对外导出`大驼峰`；单个词时用最简短的命名表示最大程度的意义，例如：i -> index、k -> key、v -> value，不过此种单字符命名一般出现在循环、条件变量中；命名中变量不要带有类型信息；保持变量声明与使用之间的距离越近越好，或者在第一次使用前再进行声明；接口名的命名一般是`方法+er`，并且在使用上，Go语言推荐尽量定义小接口，并且通过接口组合的方式构建程序。

## 声明形式

```flow
st=>start: 开始
op1=>operation: var a int32
op2=>operation: a :=17
op3=>operation: a :=(int32)17
op4=>operation: var a = 17
op5=>operation: var a = (int32)17
op6=>operation: var a int32
cond1=>condition: 包级变量?
cond2=>condition: 延迟初始化?
cond3=>condition: 是否使用默认类型?
cond4=>condition: 延迟初始化?
cond5=>condition: 是否使用默认类型?
st->cond1(yes)->cond2(yes)->op6
cond1(yes)->cond2(no)->cond3(yes)->op4
cond1(yes)->cond2(no)->cond3(no)->op5
cond1(no)->cond4(yes)->op1
cond1(no)->cond4(no)->cond5(yes)->op2
cond1(no)->cond4(no)->cond5(no)->op3
```

## 切片

切片是Go语言提供的重要且最常见的数据类型之一，可以将切片称之为`数组的描述符`，代替了数组，减少了数组指针作为函数参数的使用；append在切片上的运用让部分切片类型支持`零值可用`的理念，并且可动态扩容，减少使用者的底层存储内存分配工作；在可以预估切片容量的情况下，使用cap参数创建切片可以提升append的平均`操作性能`，减少频繁的动态扩容带来的性能损耗。

## 字典

字典也同样是Go语言提供的重要数据类型，使用中主要有几个要点需要注意：不能依赖于map的元素遍历顺序；map`不是线程安全`，`不支持并发写`（主要原因为底层数据结构`hmap中的flags`是`有状态`的，并且并没有做并发保护，由此并发写时会造成panic）；不要尝试获取map中元素（value）的地址；尽量使用`cap参数创建map`，以提升map的平均`访问性能`，减少频繁扩容带来的性能损耗。

## 求值顺序

#### 包级别变量表达式求值顺序

Go中，包级别变量的初始化按照变量声明的先后顺序进行；如果一个变量的初始化表达式中直接或间接以来其他变量，那么它的初始化顺序一定排在依赖的变量后边；未初始化的且不含有对应初始化表达式或者初始化表达式不依赖于任何未初始化变量，我们称之为`ready for initiation`变量；包级别变量的初始化是逐步进行的，每一步就是按照变量声明顺序找到下一个`ready for initiation`并对其初始化，直至没有`ready for initiation`为止。

#### 普通求值顺序

从左往右

#### 赋值语句求值顺序

先从左往右求值，后从左往右赋值

#### switch/select语句中的表达式求值

`惰性求值`即为需要对其求值时才会求值，能省则省，让计算机少做事，降低程序消耗。

## 控制语句惯用法

#### if控制语句

遵守`快乐路径`原则，即为当出现错误时，快速返回；成功逻辑不嵌入if-else语句中；`快乐路径`的执行逻辑在代码布局上始终靠左，使阅读代码时一眼就可看到正确的逻辑流程；`快乐路径`的返回值一般放在函数的最后一行。

## init函数

运行时调用、顺序、仅执行一次；init是包出场前的唯一“质检员”（张全蛋）

## 方法集合

对于非接口类型的自定义类型T，其方法集合由所有receiver为T类型的方法组成；而类型`*T`的方法集合则包含所有receiver为T和`*T`类型的方法。例：

```
type Interface interface{
	M1()
	M2()
}

type T struct{}

func (t T)	M1() {}
func (t *T)	M2() {}

func main() {
	var t T
	var pt *T
	var i Interface
	
	i = t //失败	t内只有M1方法
	i = pt //成功	pt则有M1和M2方法
}
```

在结构体类型中嵌入结构体类型为Gopher提供了一种实现“继承”的手段，**外部结构体类型T可以“继承”嵌入结构体类型的所有方法的实现**，无论是T类型的变量实例或者是*T类型变量实例，都可以调用所有“继承”的方法。例：

```
type T1 struct{}
func (T1) M1() {}
type T2 struct{}
func (T2) M2() {}

type T struct{
	T1
	*T2
}

func main() {
	t := T{
		T1: T1(),
		T2: &T2(),
	}
	T.M1()
	T.M2()
}
```

**对于基于自定义非接口创建的新类型**则没有“继承”原类型的方法集合，新类型的方法集合是空的。

**类型别名与原类型是几乎等价**的，顾名思义，就是别名而已，所以基于类型别名定义的变量也可以使用原类型对应的方法集合。

## 接口类型

提到接口类型就会想到Go的**经典问题 nil error值 ！= nil**，为什么是不等于呢，想要弄清楚这个问题，需要先了解接口类型的内部显示；

接口类型变量有两种内部表示，分别是**eface**（没有方法的空接口类型变量：interface{}）和**iface**（用于表示其他有方法的接口类型变量：interface）；二者**相同点**在于都有**两个指针字段**，并且**第二个指针字段的作用相同**，都是指向当前赋值给该接口类型变量的动态类型变量的**值**；**不同点**则在于**eface**所表示的空接口类型并**无方法列表**，因此**第一个指针类型**指向一个**_type**类型结构，该结构为该接口类型变量的动态类型信息，而**iface**除了要**存储动态类型信息**外，还要**存储接口本身信息**（类型信息、方法列表信息等）以及**动态类型所实现的方法的信息**，因此iface第一个指针字段指向itab类型结构；

```
type iface struct{
	tab	*itab
	data unsafe.Pointer
}
type eface struct{
	_type *_type
	data unsafe.Pointer
}
type _type struct{
	size	uintptr
	ptrdata	uintptr
	hash	uintptr
	tflag	uintptr
	...
}
type itab struct{
	inter *interfacetype
	_type *_type
	hash  uint32
	_ 	  [4]byte
	fun   [1]uintptr
}
```

然后回到最开始的问题，为什么 nil error值 ！= nil ？就很清楚的知道了，因为非空接口类型变量的类型信息并不为空，数据指针为空，因此它与nil（0x0,0x0）之间不能划等号。

接口类型比较时，当且仅当两个接口类型变量的类型信息（eface._type/iface.tab._type）相同，且数据指针（eface.data/iface.data）所指数据相同时，两个接口类型才是相等的。

## 错误处理策略

要写出高质量的Go代码，我们需要**始终想着错误处理**。这些年来，Go核心开发团队与Go社区已经形成了4种惯用的**Go错误处理策略**。

#### 透明错误处理策略

最简单的错误策略莫过于完全不关心返回错误值携带的具体上下文信息，只要发生错误就进入唯一的错误处理之行路径。这也是Go语言中**最简单最常见的错误处理策略**。

#### “哨兵”错误处理策略

如果不能仅根据透明错误值就做出错误处理路径的选取决策，错误处理放会尝试对返回的错误值进行检视，于是就有可能出现下面的反模式：

错误处理方以透明错误值所能提供的唯一上下文信息作为选择错误处理路径的依据，这种反模式会造成验证的**隐式耦合**：错误值构造反方不经眼间的一次错误描述字符串的变动，都会造成错误处理行为的变化，并且通过字符串比较的方式对错误值进行检视的性能也很差。

```
var ErrSentinel = errors.New("the underlying sentinel error")

func main() {
	err1 := fmt.Println("wrap err1: %w", ErrSentinel)
	err2 := fmt.Println("wrap err2: %w", err1")
	if errors.Is(err2, ErrSentinel) {
		println("err is ErrSentinel")
		return
	}
	
	println("err is not ErrSentinel")
}
```



#### 错误值类型检视策略

有时基于Go标准库提高的错误类型构造方法构造“哨兵”错误值并未提供有效的错误上下文信息，只能进行简单的错误值处理比较。

我们可以通过自定义错误类型的构造值的方式来提供更多的错误上下文信息，并且通过error接口变量统一呈现，要得到底层的错误类型携带的错误上下文信息，错误处理方需要使用Go提供的**类型断言机制**或**类型选择机制**，这种被称为**错误值类型检视策略**。

```
type MyError struct {
	e string
}

func (e *MyError) Error() string {
	return e.e
}

func main() {
	var err &MyError{
	"my error type"
	}
	err1 := fmt.Errorf("wrap err1: %w",err)
	err2 := fmt.Errorf("wrap err2: %w",err1)
	var e *MyError
	if errors.As(err2, &e) {
		println("MyError is on the chain of err2")
		println(e == err)
		return
	}
	
	println("MyError is not on the chain of err2")
}
```

#### 错误行为特征检视策略

除了透明错误处理策略，是否还有手段可以降低错误处理方与错误值构造方的耦合，在Go的标准库中发现了一种这样的错误处理方式：将莫个包中的错误类型归类，统一提取出一些公共的错误行为特征，并将这些错误行为特征放入一个公开的接口类型中。

```
type Error struct {
	error
	Timeout() bool		// 超时错误？
	Temporary() bool	// 临时错误？
}
```

## Go网络编程模型（TCP Socket）

#### TCP Socket 网络编程模型

网络I/O模型（网络编程模型）的定义是应用线程与操作系统内核之间的交互行为模式。我们通常用**阻塞**和**非阻塞**来描述网络I/O模型。不同标准对于网络I/O模型的说法各不同，比如POSIX.1标准还定义了**同步**和**异步**这两个术语来描述模型。

阻塞和非阻塞是以内核是否等数据完全就绪才返回来区分的。如果内核等到全部的数据**就绪才返回**，则这种行为模式称之为**阻塞**；如果内核查看数据就绪状态后，即使没有就绪也**立即返回**错误，则这种行为模式称之为**非阻塞**。

常见的网络I/O模型有以下几种：

**阻塞I/O模型：**最常用的模型，该模型下的应用线程和系统内核之间的交互行为模式，简述来说就是：

1. 应用线程向操作系统内核发起I/O请求
2. 内核将尝试执行此I/O操作，等待所有数据就绪后，将数据从内核空间复制到用户空间
3. 最后系统调用从内核空间返回，并返回执行成功提示

但是在这期间，用户空间的应用线程将一直阻塞在这等待，无法做其他处理；因此在这种模型下，所有的Socket默认都是阻塞的，一个线程只能处理一个网络连接上的数据通信，即便连上没数据，也只能阻塞等待在Socket的读操作上。不过虽然此模型对于应用整体来说是低效的，但是对于开发人元来，基于此模型进行开发网络应用通信是最简单的。

**非阻塞I/O模型：**和阻塞I/O模型相反，在非阻塞I/O模型下交互模式为：

1. 应用线程向操作系统内核发起I/O请求
2. 内核会尝试执行此I/O操作，如果数据并未准备就绪，系统调用则会立即返回错误状态码
3. 最后线程基于错误状态码选择如何后续操作

但是实际使用上，使用非阻塞模型的应用线程通常会通过轮询的方式一次次的发起I/O请求，直至读取所需的数据；不过这种方式对CPU的计算资源是极大的浪费，因此非阻塞I/O模型单独应用的比例并不高。

**I/O多路复用模型：**为了避免非阻塞I/O模型轮询对计算资源的浪费以及阻塞I/O模型的抵消，开发人员开始首选I/O多路复用模型作为网路I/O模型。I/O多路复用模型建立在操作系统提供的select/poll等多路复用函数（以及性能更好的epoll等函数）的基础上，具体交互逻辑为：

1. 应用线程首先将需要进行I/O操作的Socket都添加到多路复用函数（select）中，接着阻塞，等待select系统调用返回
2. 当内核发现有数据到达时，对应的Socket具备通信条件，select函数返回。
3. 应用线程再基于该Socket再次发起网络I/O请求。由于数据已经准备就绪了，即便Socket是阻塞的，第二次网络I/O请求还是非常快。

阻塞模型一个线程仅能处理一个Socket，而在I/O多路复用模型中，应用线程可以同时处理多个Socket；虽然可以同时处理多个Socket，但是I/O多路复用模型由内核实现可读/可写事件的通知，避免了非阻塞模型中轮询带来的CPU计算资源的浪费。

**异步I/O模型：**在异步I/O模型下，应用线程与系统内核的交互行为模式与前几种模型的差异化较大，具体如下：

1. 应用线程发起异步I/O调用后，内核将启动等待数据的操作并且马上返回。（之后线程可以继续执行其他操作，即无需阻塞，也无需轮询再次发起I/O调用）
2. 在内核空间数据就绪并可以被内核空间复制到用户空间后，内核会主动生成信号以驱动执行用户线程在异步I/O调用时注册的信号处理函数，或者主动执行用户线程注册的回调函数，让用户线程完成对数据的处理。

相对上述几个模型，异步I/O模型受各个平台的支持程度不一，且使用起来复杂度较高，如何进程内存管理、信号处理/回调函数等逻辑设计上会给开发人员带来不小的心智负担。

前面提到的几个模型（阻塞、非阻塞、多路复用）均可以看成同步I/O模型，都会引起线程的阻塞，直到I/O操作的完成；而只有异步I/O才是名副其实的”异步I/O“模型。

伴随着模型的演进，服务程序愈发强大，可以支持更多的连接，获得更好的处理性能。目前主流网络服务器采用的多是I/O多路复用模型，有的也结合多线程。不过I/O多路复用模型在支持更多连接、提升I/O操作效率的同时，给使用者也带来了不低的复杂性，以至于出现了高性能的多路复用框架（如libevent、libev、libuv等）以降低开发复杂性，减轻开发者的心智负担。

不过Go语言的设计者认为I/O多路复用的这种通过回调割裂控制流的模型复杂性仍然很高，为此他们结合Go语言的自身特点，将复杂性隐藏在了Go运行时中。这样，大多数情况下，Go开发者无需关心Socket是不是阻塞，也无需亲自将Socket文件描述符的回调函数注册到类似于select这样的系统调用中，而只需要在每个连接对应的goroutine中以最简单、最易用的阻塞I/O模型的方式进行Socket操作即可，可大大减轻开发心智负担。

#### TCP连接建立

建立TCP Socket连接需要经历客户端和服务端的三次握手过程。在建立连接过程中，服务端是一个标准的**Listen+Accept**的结构，而客户端是**Dial/DialTimeout**函数发起建立连接。

对客户端而言，建立连接时可能会遇到以下几种情形：

**网络不可达或服务未启动**：对于Dial的服务端地址网络不可达，或者服务地址对应的服务并没有启动的，端口未监听，Dial会立即返回错误。

**服务的listen backing队列满**：这种场景就是服务端忙，瞬间有大量客户端尝试与服务端建立连接，服务端可能会出现listen backing队列满，接受连接不及时的情况，这就可能导致客户端的Dial调用阻塞。（backlog队列的槽位数量和系统的net.ipv4.tcp_max_sync_backlog配置相关）

**网络延迟大：**如果网络延迟很大，TCP握手过程历经磨难（丢包），时间消耗自然会更长，Dial此时会发生阻塞，如果长时间阻塞的话，可能会造成客户端的请求超时；一般情况下，客户端使用Dial是可以满足交互需求的，但是如若对连接时间有着严格限定的话，可以使用DialTimeout函数。

#### Socket读写

连接建立完成后，我们就要在连接上进行读写操作以完成业务逻辑。前面说过，Go运行时隐藏了多路复用的复杂性，我们使用的时候只需要采用goroutine+阻塞I/O模型即可满足大部分场景需求。Dial连接成功后会返回一个net.Conn接口类型的变量值，这个接口变量的底层类型为一个*TCPConn：

```
type TCPConn struct{
	conn
}
```

TCPConn内嵌了一个非导出类型conn，因此继承了其类型对应的Read和Write方法，后续Dial函数返回值通过调用Write和Read方法均是net.conn的方法。

通过几个场景来总结说明下conn.Read的行为特点：

**Socket中无数据：**连接建立后，如若未进行发送数据，则接受方就会一直阻塞在Socket读操作上，这和前面提到的阻塞I/O的行为模式是一样的。执行该操作的goroutine也会被挂起，Go运行时sysmom会进行监视该Socket，直到其有数据读事件才会重新调度该Socket对应的goroutine完成读操作。

**Socket中有部分数据：**如果Socket中有部分数据准备就绪，数据长度小于期望的数据长度，那么此次读操作会执行成功并读取这部分数据。

**Socket中有足够多的数据：**如果Socket内的数据长度大于我们期望读取的数据长度，这种场景最符合我们的期望，会读取我们期望长度的数据长度，再次读取时会继续读取Socket内的数据，直至无数据。

**Socket关闭：**如果发送方关闭了Socket连接，那么接收方会读取到什么呢？这里有可分为**有数据关闭**和**无数据关闭**的场景；有数据关闭->有数据时Read操作成功，读取指定字节的数据，直至无数据，无数据时将会返回错误EOF（连接断开）；无数据关闭->无数据时将会返回错误EOF（连接断开）。总结来说：有数据读数据，无数据返回EOF。

**读操作超时：**即为无数据读取时，返回超时错误。

**成功写：**即为调用Write返回的n和error复合预期。

**写阻塞：**TCP通信连接两端的操作系统内核都会为该连接保留数据缓存区，一端调用Write后，实际上数据是写入操作系统协议栈的数据缓存区中。TCP是全双工通信，因此每个方向都有独立的数据缓存区。当发送方将对方的接受缓存区及自身的发送缓存区写满后，Write调用就会阻塞。

**写入部分数据：**Write操作存在服务方关闭后仍写入部分数据的情况。

**写入超时：**就算设置了写入写入超时，也仍然出现上述情况。

综上可知，虽然Go提供了阻塞I/O的便利，但是在调用Read和Write时仍要结合这两个方法的返回值进行正确的处理。

**goroutine安全并发读写：**goroutine的网络编程模型决定了存在不同goroutine间共享conn的情况，那么conn的读写是不是并发安全的呢？

Write操作是有锁保护的，直至数据全部写完；因此在应用层面，想要保证多个goroutine在一个conn上的write的操作是安全的，需要让一次write操作完整的写入一个业务包，一旦业务包被拆分为了多次的Write操作，就无法保证某个goroutine的业务包在conn发送数据的连贯性。

Read操作也是有锁保护的，多个goroutine对同一conn的并发读不会出现内容重叠的现象，但是因为内容断点是依运行调度来随机确定的，就可能导致，存在一个业务包被多个goroutine分开读走的情况。
