---
title: "Go插件的使用"
date: 2024-05-29

categories: [learning note]
tags: ["go"]
keywords: ["go plugin"]

description: "learning of go plugin usage"
---

## Q1:说明 go plugin 使用场景，并编写一个使用 plugin 的 demo
Go plugin支持将Go包编译为共享库（.so）的形式单独发布，主程序可以在运行时动态加载这些编译为动态共享库文件的go plugin，从中提取导出(exported)变量或函数的符号并在主程序的包中使用。Go plugin的这种特性为Go开发人员提供更多的灵活性，我们可以用之实现支持热插拔的插件系统。


### demo 通过不同插件实现io.StringWriter，可将输出保存到不同地方
#### plugin/plugin.go
```go
package main

import (
	"fmt"
	"log"
	"os"
)

func init() {
	log.Println("stdoutStringWriter init")
}

type stdoutStringWriter struct{}

func (w *stdoutStringWriter) WriteString(s string) (n int, err error) {
	return fmt.Fprint(os.Stdout, s)
}

var StringWriter stdoutStringWriter
```

#### main.go
```go
package main

import (
	"io"
	"log"
	"plugin"
)

func main() {
	log.SetFlags(log.Lshortfile | log.LstdFlags)
	p, err := plugin.Open("./plugin/plugin.so")
	if err != nil {
		panic(err)
	}
	w, err := p.Lookup("StringWriter")
	if err != nil {
		panic(err)
	}
	sw, ok := w.(io.StringWriter)
	if !ok {
		panic(err)
	}
	n, err := sw.WriteString("Hello World!\n")
	if err != nil {
		log.Println(err)
	}
	log.Println("written:", n)
}
```

#### stdout
```
$ go build -buildmode=plugin -o plugin/plugin.so plugin/plugin.go
$ go run .
2024/05/29 14:37:02 plugin.go:10: stdoutStringWriter init
Hello World!
2024/05/29 14:37:02 main.go:27: written: 13
```
plugin.go 的 `log.Println` 也受到了 main.go 的 `log.SetFlags` 影响
## Q2:说明 go plugin 使用约束
- go plugin只支持Linux, FreeBSD和macOS
- plugin包名必须为main
- 目前只能加载,不能卸载so
- 主程序与plugin的共同（间接）依赖包的版本必须一致
- 如果采用mod=vendor构建，那么主程序和plugin必须基于同一个vendor目录构建
- 主程序与plugin使用的编译器版本必须一致
- 使用plugin的主程序仅能使用动态链接

## 参考
- [https://pkg.go.dev/plugin](https://pkg.go.dev/plugin)
- [https://tonybai.com/2021/07/19/understand-go-plugin/](https://tonybai.com/2021/07/19/understand-go-plugin/)
- [https://medium.com/profusion-engineering/plugins-with-go-7ea1e7a280d3](https://medium.com/profusion-engineering/plugins-with-go-7ea1e7a280d3)