课程134：启动协程
runtime.NumCPU() 显示的是支持的线程数，由于当前CPU支持超线程技术，所以一般就是实际CPU*2
runtime.NumGoroutine 当前go进程数目


课程135：协程生命周期与waitGroup
除了main进程 其他go程是平等的，也就是说在除了main go程启动的go程不会因为开启此go程的程序结束而结束
var wt = sync.WaitGroup{}
wt.add() wt.Done()   wt.Wait()


课程136：并发安全和原子操作
n++是非原子操作
atomic.AddInt32 原子加操作
atomic.LoadInt32 原子取数
如何测试？ for循环启动go程循环+1000次 waitGroup等待所有go程结束后打印值 如果小于1000说明不安全


课程137：读写锁
var lock sync.Mutex 普通锁
var lock sync.RWMutex 读写锁  读读不互斥 读写和写写互斥


课程138：如何并行修改结构体、切片、map
结构体，切片，数组 如果并发各个修改不会有并发安全问题  如果一起改一个 可能会有问题
map 存在并发安全问题   并发写map会产生fatal error
需要用sync.map 写元素用Store  读的话用Load


课程139：读写锁和泛型的综合练习
如果有返回值写写构体的操作函数应该穿指针  写构造函数应该传指针  定义泛型放在泛型结构体后面  如写map时候 K comparable 和V any
传递泛型结构体都要加上泛型标志 
初始化map用 make(map)
课程140：recover与协程
recover只能捕获本协程内的panic


课程141：channel的阻塞与遍历
读空channel 或者 插入满的channel 会阻塞
close channel 后才能for循环读取channel （在单个线程时）  


课程142：阻塞代码的5种方法及导致死锁的根本原因
1.没人done导致的wait阻塞
2.读取空channel
3.未释放锁再次执行Lock
4.select{}
5.time.Sleep

如果没有其他go程会爆出fatal error deadlock 即当前存活的所有线程进入阻塞就会deadlock


课程143：用channel实现广播和CountDownLatch
实现广播
对于一个管道未close 读取会进入阻塞  而close后就可以读取  区分是否是关闭后读取的默认值 可以用返回布尔值判断
所以可以先读取一个channel阻塞 等待close 就实现了广播机制
实现CountDownLatch（等待多个协程结束）
channel for循环读取多次即可 其功能类似于waitGroup


课程144：招人嫌的sync.Cond
使用channel传递信号相比于sync.Cond要更好


课程145：MPG并发模型
机器（Machine）--处理器（Processor）--协程（Goroutine）
所有协程优先级是一样的 谁先执行完也完全无法控制 协程调度更轻量是因为其在用户态runtime完成 不需要和内核态交互 因此更快更轻量


课程146：协程与线程对比
python 一个进程只能有一个核来完成执行 因此开启协程也无法完全利用CPU
![alt text](img\python.png)
C++和java 进程和内核线程一对多的关系
go语言内核线程和go程式一对多的关系
![alt text](img\javaorgo.png)

协程优势： 1.创建数量高 内存占用小 2.切换成本小 非抢占式由Go runtime主动交出控制器权  3.创建销毁消耗非常小 因此java这种语言需要创建线程池

课程147：用channel并行处理海量文件
 如deal_file所写  上下游channel相互配合 关闭channel时要依次关闭 先关闭上游channel  这样下游读取就会解除阻塞 


课程148：用channel限制接口的并发请求量
进入函数插入到channel一个元素 defer拿出一个元素

课程149：用channel限制协程的总数
不在直接创建协程 而是通过执行固定的run方法
```
type GoroutineLimiter struct {
	limit int //缓冲长度为limit，运行的协程不会超过这个值
	ch    chan struct{}
}

func NewGoroutineLimiter(n int) *GoroutineLimiter {
	return &GoroutineLimiter{
		limit: n,
		ch:    make(chan struct{}, n),
	}
}

func (g *GoroutineLimiter) Run(f func()) { //函数作这参数
	g.ch <- struct{}{} //创建子协程前往管道里send一个数据
	go func() {
		f()
		<-g.ch //子协程退出时从管理里取出一个数据
	}()
}
```

课程150：select多路监听(channel)
对channel监听 那个channel有元素执行那个，假若都有元素会随机执行
select相当于多if


课程151：不使用once的单例模式





