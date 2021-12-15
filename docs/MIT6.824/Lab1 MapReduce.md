## **论文笔记**







## **实验结果**

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211208190000-fd174d3bf1f6dab2424b7c9cc7331599-MapReducePass-bad0a1.png" style="zoom:50%;" />

## **实现思路**

### **数据结构**

用一个枚举类型State表示Master, Worker, MapTask和ReduceTask当前状态, 初始状态都为Spare

```go
type State int8
const (
	Spare State = iota
	Mapping
	Reducing
	Done
)
```

**Worker**

```go
type WorkerBlock struct {
	mu 			sync.Mutex
	quit		chan bool
	id			string
	stat		State
	mapf 		func(string, string) []KeyValue
	reducef 	func(string, []string) string
}
```

**Master**

```go
type Master struct {
	mu			sync.Mutex
	masterStat	State

	nReduce 	int
	workers 	map[string]*WorkerState
	files  		[]string
	rTasks     	[]*ReduceTask
	mTasks     	[]*MapTask
	asignMap 	map[string]*MapTask
	asignReduce map[string]*ReduceTask
}
```

### **大致思路**

这个Lab没有什么算法，基本上是工程问题



Input1 -> Map -> a,1 b,1  

Input2 -> Map ->     b,1  

Input3 -> Map -> a,1     c,1                    

​                  |   |   |                    

​                  |   |   -> Reduce -> c,1                      

​                  |   -----> Reduce -> b,2                    

​                  ---------> Reduce -> a,2

最后Map得到的文件应该是 文件数量*nReduce 的一个矩阵

Reduce后会得到nReduce个文件



Worker启动时调用Master.Register RPC进行注册，添加到Master的workers Map

**Master**	

​	利用mrmaster调用MakeMaster时传进来的文件名初始化任务列表mTasks

​	新建一个goroutine执行schedule()，每deltaTime检查一次当前状态

​	处于Mapping状态时 遍历检查worker状态，如果worker空闲则遍历任务列表寻找分配；如果正在工作则检查定时器，超时则调用Worker.Stop RPC并从assignment中删除(worker, mTasks)，mTasks设为Spare。如果rpc.call返回的err != nil则判断worker已经crash，删除worker，如果err == nil则判断worker只是超时但还在工作，将worker设回Spare

​	处于Reducing状态时 细节稍有不同但基本同上

### **RPC处理函数**

**Worker.DoMap** go doMap2()新建一个goroutine执行Map任务，返回

​	func doMap2()可以直接用mrsequential.go的代码，利用os.CreateTemp，全部写操作完成后再重命名，提高fault tolerance和防止冲突文件写入

**Worker.DoReduce** 基本同上

**Worker.Stop** 设置worker为Spare，用channel通知还在执行的doMap2() goroutine已经超时了。因为doMap2()是顺序执行的不能在执行过程中停止，在最后再判断是否已经超时，超时就删除已经写入的临时文件然后退出，没超时就超时Master.ReportMapState RPC

**Worker.Shutdown** 调用os.Exit退出

**Master.Register** 注册worker，添加到workers Map

**Master.ReportMapState** 重置worker的定时器，从assignment中删除(worker, mTasks)，mTasks设为Done，worker设为Spare，记录Map得到的文件，分别添加到对应bucket的rTask中

**Master.ReportReduceState** 基本同上

### **一些细节**

对可能被多个goroutine读写的数据操作前后sync.Mutex加锁，性能有影响，不知道之后有没有更简洁完善的办法；go build或者run的时候 -race开启go的竞争检测

#### **计时器**

对worker的计时简单地用time包封装了两个函数用来新建和重置定时器

```go
func newTimer() *time.Timer {
	t := time.NewTimer(0)
	if !t.Stop() {
		<-t.C
	}
	return t
}

func (m *Master) stopResetTimer(t *time.Timer, d time.Duration) {
	if !t.Stop() {
		select {
		case <-t.C:
		default:
		}
	}
	t.Reset(d)
}
```

master的心跳检查可以用time.Tick()，返回一个<-chan Time

#### **命名**

WorkerID是一个16位随机字符串, 创建Worker时生成 

MapTaskID为sha1($filename) Master初始化文件列表时生成 

ReduceTaskID就是ihash(key) % nReduce得到的bucket

Worker Map结果文件名为"tmp-$MapTaskID-$ReduceTaskID"

Reduce完成后文件名为"mr-out-$ReduceTaskID"

## **Debug**

用Golang自带的log包, 命令行执行的时候手动重定向"2>mr.log"到文件然后把log打印的行号和代码对照着debug

test-mr.sh做的事大致就是   单独测试可以复制一份自己改   

crash.go 每次map和reduce 

## 















