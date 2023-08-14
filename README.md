<div align="center">
</br>

<img src="./logo/gopool-logo-350.png" width="120">

# GoPool

[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat&logo=github&color=2370ff&labelColor=454545)](https://makeapullrequest.com)
[![build and test](https://github.com/devchat-ai/gopool/workflows/CI/badge.svg)](https://github.com/devchat-ai/gopool/actions)
[![go report](https://goreportcard.com/badge/github.com/devchat-ai/gopool?style=flat)](https://goreportcard.com/report/github.com/devchat-ai/gopool)
[![release](https://img.shields.io/github/release/devchat-ai/gopool.svg)](https://github.com/devchat-ai/gopool/releases/)

| English | [中文](README_zh.md) |
| --- | --- |

</div>

Welcome to GoPool, **a project with 95% of the code is generated by GPT**. You can find the corresponding list of Commit and Prompt at [pro.devchat.ai](https://pro.devchat.ai).

GoPool is a **high-performance**, **feature-rich**, and **easy-to-use** worker pool library for Golang. It is designed to manage and recycle a pool of goroutines to complete tasks concurrently, improving the efficiency and performance of your applications.

<div align="center">
<img src="./logo/gopool.png" width="750">
</div>

## Performance Testing

This table shows the performance testing results for three Go libraries: GoPool, [ants](https://github.com/panjf2000/ants), and [pond](https://github.com/alitto/pond). The table includes the time it takes for each library to process 1 million tasks and the memory consumption in MB.

|     Project    | Time to Process 1M Tasks (s) | Memory Consumption (MB) |
|----------------|:----------------------------:|:-----------------------:|
| GoPool         | 1.13                         | 1.23                    |
| [ants](https://github.com/panjf2000/ants) | 1.43 | 9.49                 |
| [pond](https://github.com/alitto/pond)    | 3.51 | 1.88                 |

You can run the following commands to test the performance of GoPool, ants, and pond on your machine:

```bash
$ go test -benchmem -run=^$ -bench ^BenchmarkGoPoolWithMutex$ github.com/devchat-ai/gopool
$ go test -benchmem -run=^$ -bench ^BenchmarkAnts$ github.com/devchat-ai/gopool
$ go test -benchmem -run=^$ -bench ^BenchmarkPond$ github.com/devchat-ai/gopool
```

The results of the performance testing on my machine are as follows:

- GoPool

```bash
$ go test -benchmem -run=^$ -bench ^BenchmarkGoPoolWithMutex$ github.com/devchat-ai/gopool
goos: darwin
goarch: arm64
pkg: github.com/devchat-ai/gopool
BenchmarkGoPoolWithMutex-10    	       1	1131753125 ns/op	1966192 B/op	 13609 allocs/op
PASS
ok  	github.com/devchat-ai/gopool	1.5085s
```

- ants

```bash
$ go test -benchmem -run=^$ -bench ^BenchmarkAnts$ github.com/devchat-ai/gopool
goos: darwin
goarch: arm64
pkg: github.com/devchat-ai/gopool
BenchmarkAnts-10    	       1	1425282750 ns/op	 9952656 B/op	   74068 allocs/op
PASS
ok  	github.com/devchat-ai/gopool	1.730s
```

- pond

```bash
$ go test -benchmem -run=^$ -bench ^BenchmarkPond$ github.com/devchat-ai/gopool
goos: darwin
goarch: arm64
pkg: github.com/devchat-ai/gopool
BenchmarkPond-10    	       1	3512323792 ns/op	 1288984 B/op	   11106 allocs/op
PASS
ok  	github.com/devchat-ai/gopool	3.946s
```

## Features

- [x] **Task Queue**: GoPool uses a thread-safe task queue to store tasks waiting to be processed. Multiple workers can simultaneously fetch tasks from this queue.

- [x] **Concurrency Control**: GoPool can control the number of concurrent tasks to prevent system overload.

- [x] **Dynamic Worker Adjustment**: GoPool can dynamically adjust the number of workers based on the number of tasks and system load.

- [x] **Graceful Shutdown**: GoPool can shut down gracefully. It stops accepting new tasks and waits for all ongoing tasks to complete before shutting down when there are no more tasks or a shutdown signal is received.

- [x] **Task Error Handling**: GoPool can handle errors that occur during task execution.

- [x] **Task Timeout Handling**: GoPool can handle task execution timeouts. If a task is not completed within the specified timeout period, the task is considered failed and a timeout error is returned.

- [x] **Task Result Retrieval**: GoPool provides a way to retrieve task results.

- [x] **Task Retry**: GoPool provides a retry mechanism for failed tasks.

- [x] **Lock Customization**: GoPool supports different types of locks. You can use the built-in `sync.Mutex` or a custom lock such as `spinlock.SpinLock`.

- [ ] **Task Priority**: GoPool supports task priority. Tasks with higher priority are processed first.

## Installation

To install GoPool, use `go get`:

```bash
go get -u github.com/devchat-ai/gopool
```

## Usage

Here is a simple example of how to use GoPool with `sync.Mutex`:

```go
package main

import (
    "sync"
    "time"

    "github.com/devchat-ai/gopool"
)

func main() {
    pool := gopool.NewGoPool(100)
    defer pool.Release()

    for i := 0; i < 1000; i++ {
        pool.AddTask(func() (interface{}, error){
            time.Sleep(10 * time.Millisecond)
            return nil, nil
        })
    }
    pool.Wait()
}
```

And here is how to use GoPool with `spinlock.SpinLock`:

```go
package main

import (
    "time"

    "github.com/daniel-hutao/spinlock"
    "github.com/devchat-ai/gopool"
)

func main() {
    pool := gopool.NewGoPool(100, gopool.WithLock(new(spinlock.SpinLock)))
    defer pool.Release()

    for i := 0; i < 1000; i++ {
        pool.AddTask(func() (interface{}, error){
            time.Sleep(10 * time.Millisecond)
            return nil, nil
        })
    }
    pool.Wait()
}
```

## Dynamic Worker Adjustment

GoPool supports dynamic worker adjustment. This means that the number of workers in the pool can increase or decrease based on the number of tasks in the queue. This feature can be enabled by setting the MinWorkers option when creating the pool.

Here is an example of how to use GoPool with dynamic worker adjustment:

```go
package main

import (
    "time"

    "github.com/devchat-ai/gopool"
)

func main() {
    pool := gopool.NewGoPool(100, gopool.WithMinWorkers(50))
    defer pool.Release()

    for i := 0; i < 1000; i++ {
        pool.AddTask(func() (interface{}, error){
            time.Sleep(10 * time.Millisecond)
            return nil, nil
        })
    }
    pool.Wait()
}
```

In this example, the pool starts with 50 workers. If the number of tasks in the queue exceeds (MaxWorkers - MinWorkers) / 2 + MinWorkers, the pool will add more workers. If the number of tasks in the queue is less than MinWorkers, the pool will remove some workers.

## Task Timeout Handling

GoPool supports task timeout. If a task takes longer than the specified timeout, it will be cancelled. This feature can be enabled by setting the `WithTimeout` option when creating the pool.

Here is an example of how to use GoPool with task timeout:

```go
package main

import (
    "time"

    "github.com/devchat-ai/gopool"
)

func main() {
    pool := gopool.NewGoPool(100, gopool.WithTimeout(1*time.Second))
    defer pool.Release()

    for i := 0; i < 1000; i++ {
        pool.AddTask(func() (interface{}, error) {
            time.Sleep(2 * time.Second)
            return nil, nil
        })
    }
    pool.Wait()
}
```

In this example, the task will be cancelled if it takes longer than 1 second.

## Task Error Handling

GoPool supports task error handling. If a task returns an error, the error callback function will be called. This feature can be enabled by setting the `WithErrorCallback` option when creating the pool.

Here is an example of how to use GoPool with error handling:

```go
package main

import (
    "errors"
    "fmt"

    "github.com/devchat-ai/gopool"
)

func main() {
    pool := gopool.NewGoPool(100, gopool.WithErrorCallback(func(err error) {
        fmt.Println("Task error:", err)
    }))
    defer pool.Release()

    for i := 0; i < 1000; i++ {
        pool.AddTask(func() (interface{}, error) {
            return nil, errors.New("task error")
        })
    }
    pool.Wait()
}
```

In this example, if a task returns an error, the error will be printed to the console.

## Task Result Retrieval

GoPool supports task result retrieval. If a task returns a result, the result callback function will be called. This feature can be enabled by setting the `WithResultCallback` option when creating the pool.

Here is an example of how to use GoPool with task result retrieval:

```go
package main

import (
    "fmt"

    "github.com/devchat-ai/gopool"
)

func main() {
    pool := gopool.NewGoPool(100, gopool.WithResultCallback(func(result interface{}) {
        fmt.Println("Task result:", result)
    }))
    defer pool.Release()

    for i := 0; i < 1000; i++ {
        pool.AddTask(func() (interface{}, error) {
            return "task result", nil
        })
    }
    pool.Wait()
}
```

In this example, if a task returns a result, the result will be printed to the console.

## Task Retry

GoPool supports task retry. If a task fails, it can be retried for a specified number of times. This feature can be enabled by setting the `WithRetryCount` option when creating the pool.

Here is an example of how to use GoPool with task retry:

```go
package main

import (
    "errors"
    "fmt"

    "github.com/devchat-ai/gopool"
)

func main() {
    pool := gopool.NewGoPool(100, gopool.WithRetryCount(3))
    defer pool.Release()

    for i := 0; i < 1000; i++ {
        pool.AddTask(func() (interface{}, error) {
            return nil, errors.New("task error")
        })
    }
    pool.Wait()
}
```

In this example, if a task fails, it will be retried up to 3 times.
