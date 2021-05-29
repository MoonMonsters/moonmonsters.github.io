title: Golang --- 使用函数自定义排序
author: _Tao
tags: []
categories:
  - Golang
date: 2020-06-06 16:48:00
---
# golang - 使用函数自定义排序

有时候我们想使用和集合的自然排序不同的方法对集合进行排序。 例如，我们想按照字母的长度而不是首字母顺序对字符串排序。 

在golang自带的sort模块中, 可以先阅读下函数的代码:

`sort.Sort`

```go
func Sort(data Interface) {
	n := data.Len()
	quickSort(data, 0, n, maxDepth(n))
}
```

`sort.quickSort`

```go
func quickSort(data Interface, a, b, maxDepth int) {
	for b-a > 12 { // Use ShellSort for slices <= 12 elements
		if maxDepth == 0 {
			heapSort(data, a, b)
			return
		}
		maxDepth--
		mlo, mhi := doPivot(data, a, b)
		// Avoiding recursion on the larger subproblem guarantees
		// a stack depth of at most lg(b-a).
		if mlo-a < b-mhi {
			quickSort(data, a, mlo, maxDepth)
			a = mhi // i.e., quickSort(data, mhi, b)
		} else {
			quickSort(data, mhi, b, maxDepth)
			b = mlo // i.e., quickSort(data, a, mlo)
		}
	}
	if b-a > 1 {
		// Do ShellSort pass with gap 6
		// It could be written in this simplified form cause b-a <= 12
		for i := a + 6; i < b; i++ {
			if data.Less(i, i-6) {
				data.Swap(i, i-6)
			}
		}
		insertionSort(data, a, b)
	}
}
```

<!-- more -->

具体实现方式可以暂时忽略, 从中找到几个重点:

1. 参数是 Interface 类型
2. 需要重写Interface下的三个函数, `Len()`, `Swap()`, `Less()`

`sort.Interface`

```go
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int
	// Less reports whether the element with
	// index i should sort before the element with index j.
	Less(i, j int) bool
	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```



具体实现如下:

```go
package main

import (
	"fmt"
	"sort"
)

/**
方便实现排序功能, 自定义类型
 */
type StringSortByLength []string

/**
从sort.Sort的代码看, 需要实现这两个函数才能实现自定义的排序功能
Less函数是控制实际的自定义排序逻辑
想实现按长度排序功能, 那么就返回字符串的长度, 并在Less中使用len比较
 */
func (s StringSortByLength) Len() int {
	return len(s)
}

func (s StringSortByLength) Swap(i int, j int) {
	s[i], s[j] = s[j], s[i]
}

func (s StringSortByLength) Less(i int, j int) bool {
	return len(s[i]) < len(s[j])
}

func main() {

	types := StringSortByLength{"string", "int", "float", "float64", "int32"}
	sort.Sort(types)
	fmt.Println(types)

}

```

