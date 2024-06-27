---
title: "Go Trace"
date: 2024-05-29

categories: [learning note]
tags: ["go"]
keywords: ["go trace"]

description: "learning usage of go trace"
---

## Q1:请使用 runtime/trace 包，写一个使用trace的例子
启动两个goroutine，记录trace
```go
func main() {
	f, err := os.Create("trace.out")
	if err != nil {
		log.Fatalf("failed to create trace file: %v", err)
	}
	defer f.Close()

	if err := trace.Start(f); err != nil {
		log.Fatalf("failed to start trace: %v", err)
	}
	defer trace.Stop()

	var wg sync.WaitGroup
	wg.Add(2)
	go wait(&wg)
	go sum(&wg)
	wg.Wait()
}

func wait(wg *sync.WaitGroup) {
	defer wg.Done()
	time.Sleep(2 * time.Second)
}

func sum(wg *sync.WaitGroup) {
	defer wg.Done()
	i, s := 0, 0
	ch := time.After(time.Second)
	for {
		select {
		case <-ch:
			fmt.Printf("%d %d\n", i, s)
			return
		default:
			i++
			s += i
		}
	}
}
```
分析trace.out:
```sh
go tool trace trace.out
```

## Q2:请描述 runtime/trace 包的使用场景
1. 性能瓶颈分析：通过跟踪程序的执行路径，识别并优化性能瓶颈，例如长时间运行的函数或高频率调用的函数。
2. 并发问题调试：帮助发现并解决由于 goroutine 之间的竞争条件、死锁或资源争用引起的并发问题。
3. 程序行为分析：详细了解程序在不同时间点的行为，分析 goroutine 的创建和销毁、事件调度等。
4. I/O 操作分析：跟踪网络通信、文件读写等 I/O 操作，找出潜在的延迟问题。

## 参考
- [https://go.dev/blog/execution-traces-2024](https://go.dev/blog/execution-traces-2024)
