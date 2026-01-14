+++
title = 'Go 的 Worker Pool 模式'
date = 2025-06-07T19:10:10+08:00
draft = false
author = "scholar7r"
authorTwitter = "scholar7r"
+++

在一个 Go 的并发项目中，假设我们有无限量的数据需要并发的进行处理，如果根据数据量为每一条数据都开辟一个 Goroutine，那么显而易见，大量的 Goroutine 被创建，这样会快速消耗掉计算机的所有资源，是不可取的。Worker Pool 模式可以定义最大的 Worker 数量，可以让程序中的 Worker 去处理数据，杜绝因为无限制创建 Goroutine 而导致的资源耗尽。

<!--more-->

TaskChannel 和 ResultChannel 是两条不同的通道，它们分别负责通知任务和接收来自工人的结果汇报。例如有 10000 个数字，我要并发的检查这些数字是否为偶数，是偶数的数字要汇报给 ResultChannel，奇数的数字将其丢弃，不在乎汇报数字的先后顺序。

```go
taskChannel := make(chan int)
resultChannel := make(chan string)
```

为了使主线程在所有子线程任务执行完成之后再关闭，还需要一个 WaitGroup。

```go
wg := sync.WaitGroup
```

现在，需要让 TaskChannel 能够接收上级传递的所有任务，并且将其下发给所有 Worker。

```go
go func() {
  for i := range 10000 {
    taskChannel <- i
  }
  close(taskChannel)
}()
```

在确定 TaskChannel 已经将任务全部下方给 Worker 们之后，这个通道便不再被需要了。TaskChannel 之后是 ResultChannel，这个管道要一直等待 Worker，当所有的任务被完成之后才能被关闭。

```go
go func() {
  for i := range resultChannel {
    fmt.Println(i)
  }
}
```

每当一个任务被完成，就将这个任务的结果输出在标准输出中。两个管道都已经有了它们的雏形，现在 Worker 登场了。

```go
func Worker(
  taskChannel <-chan int,
  resultChannel chan<- string,
  wg *sync.WaitGroup,
  workerId int,
) {
  defer wg.Done()

  for v := range taskChannel {
    result := Process(v, workerId)
    resultChannel <- result
  }
}
```

Worker 需要了解需要到什么地方接收任务，到什么地方告知结果，所以可以将 Channel 作为参数传递给 Worker，Worker 知道了这两个管道之后就可以接收任务和传达结果了。每个任务完成了还需要告知主线程，`defer wg.Done` 会在执行完成时告知主线程我这个任务完成了，主线程了解了之后就会记录已经完成的状态。在 20 个 Worker 被召集的时候为每个 Worker 分配了编号，可以在向 ResultChannel 中传递结果的时候备注这个任务是由当前 Worker 来完成的。

Worker 知道所有任务的执行过程 Process，它们依照 Process 中定义了逻辑执行每一个任务。

```go
func Process(n int, workerId int) string {
 time.Sleep(time.Second * 3)
 return fmt.Sprintf("worker %d done %d", workerId, n)
}
```

通过使用 Worker Pool 这种并发模式可以控制 Goroutine 的并发量，保证不会因为过多的 Goroutine 导致资源耗尽。
