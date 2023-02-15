### MIT 6.824 LAB 1

 **1、数据结构：**

Master：记录Master task的信息，以及master当前阶段状态。

```go
type Master struct {
	TaskQueue     chan *Task          	// BlockingQueue
	TaskMeta      map[int]*MasterTask 	// 当前所有task的信息
	MasterPhase   State               	// 当先Master的阶段
	NReduce       int					// 每个文件分为n片
	InputFiles    []string				// 所以待处理文件
	Intermediates [][]string 			// 记录全局的Intermediate
}
```

Master task是由Master产生的不同状态，不同文件的task任务。

```
type MasterTask struct {
	TaskStatus    MasterTaskStatus
	StartTime     time.Time
	TaskReference *Task
}
```

```go
type MasterTaskStatus int
const (
	Idle MasterTaskStatus = iota
	InProgress
	Completed
)
```

Master task详细的task信息：

```go
type Task struct {
	Input         string
	TaskState     State
	NReducer      int
	TaskNumber    int
	Intermediates []string
	Output        string
}
```

```go
type State int
const (
	Map State = iota
	Reduce
	Exit
	Wait
)
```

**2、逻辑实现：**

首先启动一个master并且将master注册RPC，设置path建立Unix domain socket实现worker与master的RPC通信。此外master根据不同文件名创建不同的map task并将其push到通过channel实现的阻塞队列中等待worker唤醒。

启动worker，通过RPC通信请求在阻塞队列中的map task，根据状态机将其分配给mapper（或者reducer）处理。mapper通过mapf方法将文件中单词切割并计数（计数类型用string表示）。将获得的单词string通过ihash()映射到不同文件中，此时通知master，map task完成，更新master中map task状态。

master检查所有master是否全部完成，若完成则master切换到reduce状态，将map task更新为reduce task并push到阻塞队列。worker请求reduce task（worker时刻都在请求，若阻塞队列中无task，则处于waiting状态）分配给reducer做排序和计数的操作并写到本地，告诉master， reduce task完成更新状态。当所有的reduce task完成，更新master状态为Exit并退出。

##### 3、测试结果：

```shell
$ ./test-mr.sh        
*** Starting wc test.
--- wc test: PASS
*** Starting indexer test.
--- indexer test: PASS
*** Starting map parallelism test.
--- map parallelism test: PASS
*** Starting reduce parallelism test.
--- reduce parallelism test: PASS
*** Starting crash test.
--- crash test: PASS
*** PASSED ALL TESTS
```

4、参考引用：

[s09g/mapreduce-go (github.com)](https://github.com/s09g/mapreduce-go)

[MIT-6.824-arch/src/mr at master · Anarion-zuo/MIT-6.824-arch (github.com)](https://github.com/Anarion-zuo/MIT-6.824-arch/tree/master/src/mr)

