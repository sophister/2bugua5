# Go中的slice类型

`Go`语言中，有个 `slice` 这个数据类型，本来以为理解了，直到看到这个

[https://tour.go-zh.org/moretypes/10](https://tour.go-zh.org/moretypes/10)

```go
package main

import "fmt"

func main() {
	a := make([]int, 5)
	printSlice("a", a)
	b := make([]int, 0, 5)
	printSlice("b", b)
	c := b[:2]
	printSlice("c", c)
	d := c[2:5]
	printSlice("d", d)
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}
```

在线运行结果：

```shell
a len=5 cap=5 [0 0 0 0 0]
b len=0 cap=5 []
c len=2 cap=5 [0 0]
d len=3 cap=3 [0 0 0]
```

最后这一行输出，发现 `slice` 的 **容量** 也减小了，这个完全不能理解啊！

根据 [这篇文章](https://tiancaiamao.gitbooks.io/go-internals/content/zh/02.2.html) 的解释，
似乎 `slice` 的 `cap` 是从 `slice`的指向的数组中起始位置，到数组最后元素的个数。

这篇文章 [https://blog.go-zh.org/go-slices-usage-and-internals](https://blog.go-zh.org/go-slices-usage-and-internals) 讲解的很透彻：

>长度是切片引用的元素数目。容量是底层数组的元素数目（从切片指针开始）
