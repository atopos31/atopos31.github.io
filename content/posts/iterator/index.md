+++
date = '2025-07-17T15:26:44+08:00'
title = '关于Golang新增的迭代器'
+++

在Go1.18版本时，Go新增了泛型支持，开发者可以不限于内置数据类型(map slice chanel string)，来自定义泛型数据类型。

但在Go1.22之前，**range**关键字只能用来遍历map slice chanel string，直到Go1.22版本支持了int类型。

在Go1.23版本更新后，**range**加入了三种函数类型的支持，分别为
```Go
func(yield func() bool)
func(yield func(K) bool)
func(yield func(K, V) bool)
```
其中两种已经被定义在标准库[iter](https://pkg.go.dev/iter#pkg-types)中。

range现在可以遍历这三种函数类型，这三种函数分别接收一个0-2个参数并返回bool的函数类型参数，与之对应的range形式为
```Go
for range func(yield func() bool) { _ = yield } {
	//do something
}
for k := range func(yield func(k int) bool) { _ = yield } {
	_ = k
	//do something
}
for k, v := range func(yield func(k, v int) bool) { _ = yield } {
	_ = k
	_ = v
	//do something
}
```

上面的代码在Go1.23中可以顺利通过编译，你可能会很困惑，这样做有什么用？
## yield函数
```go
func iter2(yield func(int, int) bool) {
	for i := range 2 {
		if !yield(i, i+1) {
			return
		}
	}
}

func main() {
	for k, v := range iter2 {
		fmt.Println("iter2", k, v)
	}
}
```
执行上面的代码，输出为
```
iter2 0 1
iter2 1 2
```
我们可以发现，在iter2内调用yield函数传入的两个参数被我们写的fmt.Println("iter2", k, v)打印出来了。
如果我们不使用range，想要实现同样的效果也是可以的，像这样
```go
func iter2(yield func(int, int) bool) {
	for i := range 2 {
		if !yield(i, i+1) {
			return
		}
	}
}

func main() {
	iter2(func(k, v int) bool {
		fmt.Println("iter2", k, v)
		return true
	})
}
```
但是这样看起来远不如使用range清晰易读，实际上这也是Go编译器在做的事(类似语法糖)，当在range内使用break时，编译生成的yield函数会对应添加return false，continue则会添加return true，如果既不break也不continue，也会在末尾加入一个return true，就像上面一样。例如
```go
func iter5(yield func(int, int) bool) {
	for i := range 5 {
		if !yield(i, i+1) {
			return
		}
	}
}

func main() {
	iter5(func(k, v int) bool {
		fmt.Println("iter2", k, v)
		if k == 2 {
			return false
		}
		return true
	})
	
	// 等同于
	for k, v := range iter5 {
		fmt.Println("iter2", k, v)
		if k == 2 {
			break
		}

	}
}
```
仔细看，上面的iter5函数中是我们手动去判断yield函数的返回值来实现range内部continue和break的效果的，如果我不判断呢？
```go
func iter5(yield func(int, int) bool) {
	for i := range 5 {
		yield(i, i+1)
	}
}

func main() {
	iter5(func(k, v int) bool {
		fmt.Println("iter2", k, v)
		if k == 2 {
			return false
		}
		return true
	})

	// 等同于
	for k, v := range iter5 {
		fmt.Println("iter2", k, v)
		if k == 2 {
			break
		}

	}
}
```
输出为
```
iter2 0 1
iter2 1 2
iter2 2 3
iter2 3 4
iter2 4 5
iter2 0 1
iter2 1 2
iter2 2 3
panic: runtime error: range function continued iteration after function for loop body returned false

goroutine 1 [running]:
main.main-range1(...)
	/tmp/sandbox542296131/prog.go:23
main.iter5(...)
	/tmp/sandbox542296131/prog.go:9
main.main()
	/tmp/sandbox542296131/prog.go:23 +0x1b3
```
可见Go编译器并不是简单的将函数编译处理，同时也加入了运行时检查，如果不去检查yield返回值，会直接panic掉。
## 用法
为我们自定义的Set类型实现迭代器而无需对外暴露内部实现
```go
package main

import (
	"fmt"
	"iter"
)

// Set holds a set of elements.
type Set[E comparable] struct {
	m map[E]struct{}
}

// New returns a new [Set].
func New[E comparable]() *Set[E] {
	return &Set[E]{m: make(map[E]struct{})}
}

// Add adds an element to a set.
func (s *Set[E]) Add(v E) {
	s.m[v] = struct{}{}
}

func (s *Set[E]) All() iter.Seq[E] {
	return func(yeild func(e E) bool) {
		for k := range s.m {
			if !yeild(k) {
				return
			}
		}
	}
}

func main() {
	set := New[string]()
	set.Add("hello")
	set.Add("hello")
	set.Add("world")

	for v := range set.All() {
		fmt.Println(v)
	}
}
```
更优雅遍历Scanner
```go
var pr *io.PipeReader
for chunk, err := range ScannerToken(logReader) {
	if err != nil {
		slog.Error("log reader error", "error", err)
		break
	}
	fmt.Println(chunk)
}

func ScannerToken(reader *bufio.Scanner) iter.Seq2[string, error] {
	return func(yield func(string, error) bool) {
		for reader.Scan() {
			chunk := reader.Text()
			if chunk == "" {
				continue
			}
			if !yield(chunk, nil) {
				return
			}
		}
		yield("", reader.Err())
	}
}
```