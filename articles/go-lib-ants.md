## GO 库 - ants

### 简介

github仓库地址：https://github.com/panjf2000/ants

文档地址：https://pkg.go.dev/github.com/panjf2000/ants/v2

ants 是一个高性能且低损耗的 goroutine 池，实现了对大规模 goroutine 的调度管理、goroutine 复用，允许使用者在开发并发程序的时候限制 goroutine 数量，复用资源，达到更高效执行任务的效果。

功能特性：

* 自动调度海量的 goroutines，复用 goroutines
* 定期清理过期的 goroutines，进一步节省资源
* 提供了大量有用的接口：任务提交、获取运行中的 goroutine 数量、动态调整 Pool 大小、释放 Pool、重启 Pool
* 优雅处理 panic，防止程序崩溃
* 资源复用，极大节省内存使用量；在大规模批量并发任务场景下比原生 goroutine 并发具有更高的性能
* 非阻塞机制

写 go 并发程序的时候如果程序会启动大量的 goroutine，势必会消耗大量的系统资源（内存，CPU），通过使用 ants，可以实例化一个 goroutine 池，复用 goroutine，节省资源，提升性能。

但 ants 并不保证提交的任务被执行的顺序，执行的顺序也不是和提交的顺序保持一致，因为在 ants 是并发地处理所有提交的任务，提交的任务会被分派到正在并发运行的 workers 上去，因此那些任务将会被并发且无序地被执行。

以下是原作者对于原生 goroutine 和 ants 进行的吞吐量和内存消耗的 benchmark 测试。使用 ants 在吞吐性能上相较于原生 goroutine 有 2-6 倍的性能优势，而内存消耗则有 10-20 倍的节省优势。

![goroutine_ants_benchmark](https://user-images.githubusercontent.com/7496278/63449727-3ae6d400-c473-11e9-81e3-8b3280d8288a.gif)

### 使用

#### 安装

使用 go get 将 ants 包下载到 GOPATH 指定的目录下。

```shell
go get -u github.com/panjf2000/ants/v2
```

#### 常用方法

这里介绍了协程池最基本常用的创建、提交任务、释放等方法。

```go
// 创建一个协程池，指定容量大小
p, _ := ants.NewPool(10000)

// 提交任务
ants.Submit(func(){})

// 动态调整协程池容量
pool.Tune(1000)

// 释放协程池
pool.Release()

// 重启协程池
pool.Reboot()
```

#### 代码示例：使用 Submit 函数提交任务

使用示例如下，引用自官方仓库 README 文件的使用示例。

创建一个协程池，且设置协程池的容量为0，表示不限制容量。将任务函数 demoFunc 通过 Submit 方法进行任务提交，连续提交1000次任务，然后用 WaitGroup 等待所有任务提交。

```go
import (
	"fmt"
	"sync"
	"time"

	"github.com/panjf2000/ants/v2"
)

func demoFunc() {
	time.Sleep(10 * time.Millisecond)
	fmt.Println("Hello World!")
}

func main() {
	runTimes := 1000

	var wg sync.WaitGroup
	syncCalculateSum := func() {
		demoFunc()
		wg.Done()
	}

	p, _ := ants.NewPool(0)
	defer p.Release()
	for i := 0; i < runTimes; i++ {
		wg.Add(1)
		_ = p.Submit(syncCalculateSum)
	}
	wg.Wait()
	fmt.Printf("running goroutines: %d\n", p.Running())
	fmt.Printf("finish all tasks.\n")
}
```

执行结果如下，由于任务函数中将会睡眠10毫秒，因此等到所有任务提交且 WaitGroup 结束阻塞等待后，调用 ants.Running() 查看仍在执行的协程数量将会是1000，这些协程还在睡眠等待而未返回销毁协程。

```
...
Hello World!
Hello World!
Hello World!
Hello World!
running goroutines: 1000
finish all tasks.
```

#### 代码示例：创建指定任务函数的协程池

创建一个协程池并指定任务函数，且设置协程池的容量为10。每次循环调用 Invoke 方法获取一个 worker，并用传入的参数作为执行任务的参数。

```go
import (
	"fmt"
	"sync"
	"sync/atomic"

	"github.com/panjf2000/ants/v2"
)

var sum int32

func myFunc(i interface{}) {
	n := i.(int32)
	atomic.AddInt32(&sum, n)
	fmt.Printf("run with %d\n", n)
}

func main() {
	runTimes := 1000

	var wg sync.WaitGroup

	p, _ := ants.NewPoolWithFunc(10, func(i interface{}) {
		myFunc(i)
		wg.Done()
	})
	defer p.Release()
	for i := 0; i < runTimes; i++ {
		wg.Add(1)
		_ = p.Invoke(int32(i))
	}
	wg.Wait()
	fmt.Printf("running goroutines: %d\n", p.Running())
	fmt.Printf("finish all tasks, result is %d\n", sum)
}
```

执行结果如下，可以看到由于协程池大小小于任务数量，每次取一个可用的 worker 来执行，每次执行程序的任务顺序将不确定，且同一时间限制了最多只会有10个协程在工作。这个示例程序是计算从0加到1000的总和，因此多次执行的结果之和是确定的。

```
run with 4
run with 0
run with 6
run with 7
...
run with 977
run with 980
run with 979
run with 981
running goroutines: 10
finish all tasks, result is 499500
```

#### 代码示例：创建 MultiPool

创建 MultiPool 来执行任务，MultiPool 内部包含一个协程池数组，定义一个策略算法来从这些协程池中获取 worker，目前支持的算法有RoundRobin算法（轮询）和最少任务算法。

MultiPool 同样既可以通过 Submit 方法调用指定任务函数，也可以在创建的时候指定任务函数，通过 Invoke 方法来调用。

```go
import (
	"fmt"
	"sync"
	"time"

	"github.com/panjf2000/ants/v2"
)

func demoFunc() {
	time.Sleep(10 * time.Millisecond)
	fmt.Println("Hello World!")
}

func main() {
	runTimes := 1000

	var wg sync.WaitGroup
	syncCalculateSum := func() {
		demoFunc()
		wg.Done()
	}
	mp, _ := ants.NewMultiPool(10, -1, ants.RoundRobin)
	defer mp.ReleaseTimeout(5 * time.Second)
	for i := 0; i < runTimes; i++ {
		wg.Add(1)
		_ = mp.Submit(syncCalculateSum)
	}
	wg.Wait()
	fmt.Printf("running goroutines: %d\n", mp.Running())
	fmt.Printf("finish all tasks.\n")
}
```

#### 默认协程池

为了方便使用，很多 Go 库都喜欢提供其核心功能类型的一个默认实现。可以直接通过库提供的接口调用，如 net/http 和 ants。

ants 定义了一个默认的协程池，默认容量为 MaxInt32，上面代码示例都是创建一个协程池并调用其方法，对于默认协程池可以直接调用 ants 的同名函数。

默认协程池也需要调用 Release 函数。

```go
import (
	"fmt"
	"sync"
	"time"

	"github.com/panjf2000/ants/v2"
)

func demoFunc() {
	fmt.Println("Hello World!")
}

func main() {
	defer ants.Release()

	runTimes := 1000

	var wg sync.WaitGroup
	syncCalculateSum := func() {
		demoFunc()
		wg.Done()
	}
	for i := 0; i < runTimes; i++ {
		wg.Add(1)
		_ = ants.Submit(syncCalculateSum)
	}
	wg.Wait()
}
```

#### 配置选项

ants 支持在 NewPool、NewPoolWithFunc 创建协程池时设置配置选项，定制化协程池。

如下示例在创建协程池时，调用了 ants.WithPanicHandler 函数和 ants.WithExpiryDuration 函数，分别指定了发生 panic 时的处理函数和设定每个 worker 的超时时间。

```go
import (
	"fmt"
	"os"
	"sync"
	"sync/atomic"
	"time"

	"github.com/panjf2000/ants/v2"
)

var sum int32

func myFunc(i interface{}) {
	n := i.(int32)
	atomic.AddInt32(&sum, n)
	fmt.Printf("run with %d\n", n)
}

func panicHandler(err interface{}) {
	fmt.Fprintln(os.Stderr, err)
}

func main() {
	defer ants.Release()

	runTimes := 1000

	var wg sync.WaitGroup

	p, _ := ants.NewPoolWithFunc(10, func(i interface{}) {
		myFunc(i)
		wg.Done()
	}, ants.WithPanicHandler(panicHandler), ants.WithExpiryDuration(time.Second))
	defer p.Release()
	for i := 0; i < runTimes; i++ {
		wg.Add(1)
		_ = p.Invoke(int32(i))
	}
	wg.Wait()
	fmt.Printf("running goroutines: %d\n", p.Running())
	fmt.Printf("finish all tasks, result is %d\n", sum)
}
```

ants 提供的所有选项如下，以下是 Options 结构体的定义。

```go
type Options struct {
  ExpiryDuration time.Duration
  PreAlloc bool
  MaxBlockingTasks int
  Nonblocking bool
  PanicHandler func(interface{})
  Logger Logger
}
```

各个选项的含义如下：

* ExpiryDuration：过期时间，表示 goroutine 空闲多长时间后会被 ants 回收。
* PreAlloc：预分配，创建协程池后预分配 worker 切片。
* MaxBlockingTasks：最大阻塞任务数量，当协程池中 goroutine 数量已经达到池容量，且都处于繁忙状态，新来的任务将会阻塞等待，这个选项设置了等待任务的最大数量，超过后的新的任务将失败返回而不是阻塞。0 表示不限制阻塞队伍数量。
* Nonblocking：协程池是否阻塞，默认是阻塞。设置为非阻塞时，新任务提交时无可用 goroutine 将直接返回失败。
* PanicHandler：panic 处理，设置 goroutine 在 panic 时的处理函数。
* Logger：指定日志记录器。

ants 定义了一些名称为 With 开头的函数来设置这些选项：

```go
func WithOptions(options Options) Option {}

func WithExpiryDuration(expiryDuration time.Duration) Option {}

func WithPreAlloc(preAlloc bool) Option {}

func WithMaxBlockingTasks(maxBlockingTasks int) Option {}

func WithNonblocking(nonblocking bool) Option {}

func WithPanicHandler(panicHandler func(interface{})) Option {}

func WithLogger(logger Logger) Option {}
```

### 源码分析

ants主要的结构体与函数调用流程图如下：

![ants_flow_graph](https://blog-1304941664.cos.ap-guangzhou.myqcloud.com/article_material/go/ants_flow_graph.png)

#### 常量与默认协程池

ants 在 ants.go 中定义了一些常量、一些异常全局变量、一个默认的协程池，以及对默认协程池的操作函数。

```go
const (
	// DefaultAntsPoolSize is the default capacity for a default goroutine pool.
	DefaultAntsPoolSize = math.MaxInt32

	// DefaultCleanIntervalTime is the interval time to clean up goroutines.
	DefaultCleanIntervalTime = time.Second
)

const (
	// OPENED represents that the pool is opened.
	OPENED = iota

	// CLOSED represents that the pool is closed.
	CLOSED
)

var (
	// Init an instance pool when importing ants.
	defaultAntsPool, _ = NewPool(DefaultAntsPoolSize)
)
```

#### Pool

通过上面的代码示例，可以知道 ants 创建协程池有 ants.NewPool 和 ants.NewPoolWithFunc 两种，它们分别为提交任务时传递执行的函数，以及预先指定执行函数。它们对应的的结构体分别为 Pool 和 PoolWithFunc，下面我们先来介绍 Pool，后面再和 PoolWithFunc 做比较。

该文件定义了结构体 Pool，用来表示一个协程池，它可以接收多个任务并并发地执行它们，同时在一个可重复使用的协程池中限制执行的协程数量。

```go
type Pool struct {
	capacity int32
	running int32
	lock sync.Locker
	workers workerQueue
	state int32
	cond *sync.Cond
	workerCache sync.Pool
	waiting int32
	options *Options

	purgeDone int32
	stopPurge context.CancelFunc

	ticktockDone int32
	stopTicktock context.CancelFunc

	now atomic.Value
}
```

各个字段含义如下：

* capacity：池容量，表示该协程池最多能创建的 goroutine 数量。如果为负数，则表示容量无限制。
* running：正在执行的 goroutine 的数量。
* lock：锁，用于保护 worker 队列。ants 自己实现了一个自旋锁，用于同步并发操作。
* workers：存放一组 worker 对象的容器，可能是栈或循环队列。
* state：记录协程池当前的状态是否已关闭。
* cond：条件变量，处理任务等待和唤醒。
* workerCache：使用 sync.Pool 对象池管理和创建 worker 对象。
* waiting：执行 Submit 阻塞的协程数量。
* options：协程池配置变量结构体。
* purgeDone：清除过期 worker 的终结标记。
* ticktockDone：定时更新协程池当前时间的终结标记。
* now：协程池当前时间。

ants 的每个任务都是由 worker 对象来处理的，每个 worker 对象会创建一个对应的 goroutine 来处理任务，对应的结构体为 goWorker。

函数 NewPool 用来创建一个 Pool 对象，以下省略了一些处理代码。初始化 Pool 对象，设置容量，设置配置选项，创建一个自旋锁来初始化 lock 字段，设置了 workerCache 新建 worker 对象的方法，使用 p.lock 锁创建一个条件变量 p.cond。

```go
func NewPool(size int, options ...Option) (*Pool, error) {
	if size <= 0 {
		size = -1
	}
	opts := loadOptions(options...)
	p := &Pool{
		capacity: int32(size),         // 设置容量
		lock:     syncx.NewSpinLock(), // 创建一个自旋锁来初始化 lock 字段
		options:  opts,                // 设置配置选项
	}
	p.workerCache.New = func() interface{} { // 设置了 workerCache 新建 worker 对象的方法
		return &goWorker{
			pool: p,
			task: make(chan func(), workerChanCap),
		}
	}
	if p.options.PreAlloc { // 初始化worker数组
		if size == -1 {
			return nil, ErrInvalidPreAllocSize
		}
		p.workers = newWorkerQueue(queueTypeLoopQueue, size)
	} else {
		p.workers = newWorkerQueue(queueTypeStack, 0)
	}

	p.cond = sync.NewCond(p.lock) // 使用 p.lock 锁创建一个条件变量 p.cond

	p.goPurge()    // 启动一个协程定期清理过期worker
	p.goTicktock() // 启动一个协程定期更新协程池的当前时间

	return p, nil
}
```

#### worker

Pool.workers 字段是 workerQueue 接口类型，表示一个 worker 容器。

```go
type workerQueue interface {
	len() int
	isEmpty() bool
	insert(worker) error
	detach() worker
	refresh(duration time.Duration) []worker
	reset()
}
```

每个方法的含义：

* len：worker 数量。
* isEmpty：worker 数量是否为0。
* insert：goroutine 任务执行结束后将 worker 插入。
* detach：取出一个 worker。
* refresh：清除过期 worker 并返回它们。
* reset：重置容器。

workerQueue 有两种实现，分别为 workerStack 和 loopQueue，具体使用哪个实现根据创建 Pool 时的选项 PreAlloc 决定，设置了 PreAlloc 则使用  loopQueue 实现，否则使用 workerStack 实现。

```go
func newWorkerQueue(qType queueType, size int) workerQueue {
	switch qType {
	case queueTypeStack:
		return newWorkerStack(size)
	case queueTypeLoopQueue:
		return newWorkerLoopQueue(size)
	default:
		return newWorkerStack(size)
	}
}
```

##### workerStack

workerStack 实现基于栈，定义如下。栈结构满足后进先出的特点，因此插入的元素将追加至切片尾部，取出元素从切片尾部取。

```go
type workerStack struct {
	items  []worker // 空闲worker
	expiry []worker // 过期worker
}

func newWorkerStack(size int) *workerStack {
	return &workerStack{
		items: make([]worker, 0, size),
	}
}
```

goroutine 完成任务之后，协程池会调用 insert 方法，将相应的 worker 放回。

```go
func (wq *workerStack) insert(w worker) error {
	wq.items = append(wq.items, w) // 将元素追加至切片尾部
	return nil
}
```

新任务到来时，会调用 detach 方法从 workerStack 取出一个空闲的 worker。

```go
func (wq *workerStack) detach() worker {
	l := wq.len()
	if l == 0 {
		return nil
	}

	w := wq.items[l-1] // 从切片尾部取出元素
	wq.items[l-1] = nil // avoid memory leaks
	wq.items = wq.items[:l-1]

	return w
}
```

定时清除过期的 worker，由于栈后进先出的特性，早过期的 worker 将排在数组的前面。

```go
func (wq *workerStack) refresh(duration time.Duration) []worker {
	n := wq.len()
	if n == 0 {
		return nil
	}

	expiryTime := time.Now().Add(-duration)
	index := wq.binarySearch(0, n-1, expiryTime) // 二分法搜索空闲worker，找到过期worker界限对应的下标

	wq.expiry = wq.expiry[:0]
	if index != -1 {
		wq.expiry = append(wq.expiry, wq.items[:index+1]...) // 将index前面的空闲worker追加至过期worker
		m := copy(wq.items, wq.items[index+1:]) // 空闲worker保留index后面的，内存复制
		for i := m; i < n; i++ { // 清空后面的脏数据
			wq.items[i] = nil
		}
		wq.items = wq.items[:m]
	}
	return wq.expiry
}
```

##### loopQueue

loopQueue 实现基于循环队列，定义如下。循环队列包含一个长度为 size 的切片，head 表示队列头指针，tail 表示队列尾指针，也就是最后一个元素的下一位，isFull 表示队列是否已满，当 head 和 tail 指向同一位置时用于区分队列是空还是满。

```go
type loopQueue struct {
	items  []worker
	expiry []worker
	head   int
	tail   int
	size   int
	isFull bool
}

func newWorkerLoopQueue(size int) *loopQueue {
	return &loopQueue{
		items: make([]worker, size),
		size:  size,
	}
}
```

当创建完成后，开始状态如下，head 和 tail 指向同一个位置。

![ants_loop_queue_1](https://blog-1304941664.cos.ap-guangzhou.myqcloud.com/article_material/go/ants_loop_queue_1.png)

添加元素时，tail 指针往后移一位。取出元素时，head 指针往后移一位。

![ants_loop_queue_2](https://blog-1304941664.cos.ap-guangzhou.myqcloud.com/article_material/go/ants_loop_queue_2.png)

head 指针或 tail 指针到切片尾部需要回绕回来，此时 head 下标大于 tail。

![ants_loop_queue_3](https://blog-1304941664.cos.ap-guangzhou.myqcloud.com/article_material/go/ants_loop_queue_3.png)

当 tail 指针赶上 head 指针，则表示队列已经满了。

![ants_loop_queue_4](https://blog-1304941664.cos.ap-guangzhou.myqcloud.com/article_material/go/ants_loop_queue_4.png)

而当 head 指针赶上 tail 指针时，则表示队列空了。这两种情况用 isFull 来区分。

![ants_loop_queue_5](https://blog-1304941664.cos.ap-guangzhou.myqcloud.com/article_material/go/ants_loop_queue_5.png)

下面我们来看下源码的实现。

goroutine 完成任务之后，协程池会调用 insert 方法，将相应的 worker 放回循环队列。

```go
func (wq *loopQueue) insert(w worker) error {
	if wq.size == 0 {
		return errQueueIsReleased
	}

	if wq.isFull {
		return errQueueIsFull
	}
	wq.items[wq.tail] = w // 将元素插入队尾
	wq.tail = (wq.tail + 1) % wq.size // tail 往后移动一位，如果到达切片尾部则绕回首位

	if wq.tail == wq.head { // 判断是否已满
		wq.isFull = true
	}

	return nil
}
```

新任务到来时，会调用 detach 方法从循环队列取出一个空闲的 worker。

```go
func (wq *loopQueue) detach() worker {
	if wq.isEmpty() {
		return nil
	}

	w := wq.items[wq.head] // 取出队首元素
	wq.items[wq.head] = nil
	wq.head = (wq.head + 1) % wq.size // head 往后移动一位，如果到达切片尾部则绕回首位

	wq.isFull = false

	return w
}
```

定时清除过期的 worker，由于队列先进先出的特性，早过期的 worker 将排在队列的前面。

```go
func (wq *loopQueue) refresh(duration time.Duration) []worker {
	expiryTime := time.Now().Add(-duration)
	index := wq.binarySearch(expiryTime) // 二分法搜索空闲worker，找到过期worker界限对应的下标
	if index == -1 {
		return nil
	}
	wq.expiry = wq.expiry[:0]

	if wq.head <= index {
		wq.expiry = append(wq.expiry, wq.items[wq.head:index+1]...) // 将head到index之间的空闲worker追加至过期worker
		for i := wq.head; i < index+1; i++ {
			wq.items[i] = nil
		}
	} else {
		wq.expiry = append(wq.expiry, wq.items[0:index+1]...) // 将index到切片末尾的空闲worker追加至过期worker
		wq.expiry = append(wq.expiry, wq.items[wq.head:]...) // 将0到head的空闲worker追加至过期worker
		for i := 0; i < index+1; i++ {
			wq.items[i] = nil
		}
		for i := wq.head; i < wq.size; i++ {
			wq.items[i] = nil
		}
	}
	head := (index + 1) % wq.size // 重新调整head位置
	wq.head = head
	if len(wq.expiry) > 0 {
		wq.isFull = false
	}

	return wq.expiry
}
```

##### goWorker

Pool 的 worker 使用结构体 goWorker 表示。

```go
type goWorker struct {
	pool *Pool         // 所属协程池的引用
    task chan func()   // 任务通道，通过这个通道将类型为func()的函数作为任务发送给goWorker
	lastUsed time.Time // 记录goWorker被放回协程池的时间
}
```

run 方法启动一个协程，用来执行指定的函数。从task通道中不断地接收任务，获取函数变量后直接执行，然后将goWorker对象放回协程池。这个 for 循环将一直从task通道接收任务，直至通道关闭或取出nil任务才会终止，期间协程将一直保持运行。也就是说，每个 goWorker 只会启动一次协程，后续可以重复利用这个协程，这就是 ants 高性能的关键所在。

goWorker 每次执行一个任务，然后就会放回协程池。

```go
func (w *goWorker) run() {
	w.pool.addRunning(1)
	go func() {
		defer func() {
			w.pool.addRunning(-1)
			w.pool.workerCache.Put(w) // 将goWorker对象放回sync.Pool池
			if p := recover(); p != nil {
                // ...
			}
			w.pool.cond.Signal() // 通知已有空闲goWorker
		}()
		for f := range w.task { // 从task通道接收任务
			if f == nil {
				return
			}
			f() // 执行函数
			if ok := w.pool.revertWorker(w); !ok { // 将goWorker对象放回协程池
				return
			}
		}
	}()
}
```

#### 提交任务

当我们创建了一个协程池，我们可以调用 Submit 方法来提交任务。

```go
func (p *Pool) Submit(task func()) error {
	if p.IsClosed() {
		return ErrPoolClosed
	}

	w, err := p.retrieveWorker() // 获取一个空闲的worker
	if w != nil {
		w.inputFunc(task) // 将任务函数发送到worker的任务通道
	}
	return err
}
```

retrieveWorker 方法用于获取一个可用的 worker。

首先尝试从 workers 中获取一个可用的 worker，成功则返回 worker 进行任务处理。否则，判断是否还没用完，即正在运行的 worker 数量小于协程池容量，从 workerCache 中新建一个 goWorker，并执行其 run 方法。如果协程池容量已经用完，则判断是否是非阻塞模式，或阻塞模式但阻塞任务数量大于设置的值，是则返回 nil，表示无法获取到 worker。最后进入阻塞等待，直至可用的worker放回池中，重新循环尝试获取一个可用的 worker。

```go
func (p *Pool) retrieveWorker() (w worker, err error) {
	p.lock.Lock()

retry:
	if w = p.workers.detach(); w != nil { // 尝试从workers中获取一个可用的worker
		p.lock.Unlock()
		return
	}

	if capacity := p.Cap(); capacity == -1 || capacity > p.Running() { // 如果正在运行的worker数量小于协程池容量
		p.lock.Unlock()
		w = p.workerCache.Get().(*goWorker) // 从sync.Pool对象池获取新的worker
		w.run()
		return
	}

	if p.options.Nonblocking || (p.options.MaxBlockingTasks != 0 && p.Waiting() >= p.options.MaxBlockingTasks) { // 如果是非阻塞模式，或阻塞模式但阻塞任务数量大于设置的值，直接返回nil
		p.lock.Unlock()
		return nil, ErrPoolOverload
	}

	p.addWaiting(1)
	p.cond.Wait() // 阻塞等待，直至可用的worker放回池中
	p.addWaiting(-1)

	if p.IsClosed() {
		p.lock.Unlock()
		return nil, ErrPoolClosed
	}

	goto retry
}
```

流程图如下：

![ants_submit_chart](https://blog-1304941664.cos.ap-guangzhou.myqcloud.com/article_material/go/ants_submit_chart.png)

#### 定期清理过期worker

在 NewPool 函数中的最后会启动一个协程定期清理过期worker。

```go
go p.purgeStaleWorkers(ctx)
```

这个方法首先会启动一个定时器，时间间隔为配置的 ExpiryDuration，如果没有配置该选项，则采用默认值 1s。

定时清除过期 worker 并返回它们，然后对这些过期 worker 依次向task通道发送nil值，让worker返回。

当判断到所有worker都被清理了，需要进行一次广播，唤醒阻塞等待worker的任务。

```go
func (p *Pool) purgeStaleWorkers(ctx context.Context) {
	ticker := time.NewTicker(p.options.ExpiryDuration) // 定时器
	defer func() {
		ticker.Stop()
		atomic.StoreInt32(&p.purgeDone, 1)
	}()

	for {
		select {
		case <-ctx.Done():
			return
		case <-ticker.C:
		}

		if p.IsClosed() {
			break
		}

		var isDormant bool
		p.lock.Lock()
		staleWorkers := p.workers.refresh(p.options.ExpiryDuration) // 清除过期 worker 并返回它们
		n := p.Running()
		isDormant = n == 0 || n == len(staleWorkers)
		p.lock.Unlock()

		for i := range staleWorkers {
			staleWorkers[i].finish() // 向task通道发送nil值，让worker返回
			staleWorkers[i] = nil
		}

		if isDormant && p.Waiting() > 0 { // 当所有worker都被清理了，需要广播唤醒阻塞等待worker的任务
			p.cond.Broadcast()
		}
	}
}
```

#### PoolWithFunc

上面提到过，协程池分为创建后通过 Submit 提交任务函数的 Pool，以及创建时指定任务函数的 PoolWithFunc。PoolWithFunc 对应的 worker 结构体为 goWorkerWithFunc。以下是它们的结构体：

```go
type PoolWithFunc struct {
	workers workerQueue        // 使用goWorkerWithFunc实现
	poolFunc func(interface{}) // 保存执行函数
    // 其他字段和Pool相同
}

type goWorkerWithFunc struct {
	pool *PoolWithFunc
	args chan interface{}
	lastUsed time.Time
}
```

PoolWithFunc 和 Pool 的区别在于保存了传入的函数对象，而 goWorkerWithFunc 以 interface{} 表示参数作为通道类型，而非函数作为通道类型。

#### 自旋锁

自旋锁是 Linux 内核中一种较为常见的锁机制，一般的锁在加锁失败时会休眠等待，以让出CPU处理其它事情，而自旋锁则是会忙等待，避免加锁和解锁导致的线程切换，对于很快就能获得锁的场景可以提升性能。

ants 采用 atomic.CompareAndSwapUint32() 这个原子操作实现了一个自旋锁。这里使用了指数退避，先等1个循环周期，通过 runtime.Gosched() 让运行时切换其它协程运行。如果还是获取不到锁，再等2、4、8、16个周期。防止短时间内获取不到锁导致CPU时间浪费。

```go
type spinLock uint32

const maxBackoff = 16

func (sl *spinLock) Lock() {
	backoff := 1
	for !atomic.CompareAndSwapUint32((*uint32)(sl), 0, 1) {
		// Leverage the exponential backoff algorithm, see https://en.wikipedia.org/wiki/Exponential_backoff.
		for i := 0; i < backoff; i++ {
			runtime.Gosched()
		}
		if backoff < maxBackoff {
			backoff <<= 1
		}
	}
}

func (sl *spinLock) Unlock() {
	atomic.StoreUint32((*uint32)(sl), 0)
}

// NewSpinLock instantiates a spin-lock.
func NewSpinLock() sync.Locker {
	return new(spinLock)
}
```

