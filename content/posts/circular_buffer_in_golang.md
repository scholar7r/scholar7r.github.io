+++
title = '使用 Golang 实现环形缓冲区'
date = 2025-09-03T17:48:36+08:00
draft = false
+++
环形缓冲区 Circular Buffer 也称之为圆形队列或循环缓冲区。用于表示一个固定尺寸，头尾相连的缓冲区数据结构，适合用用于缓存数据流。

环形缓冲区的特性是：当前方的一个元素被用掉之后，其余的数据元素不需要移动其位置。相反一个非环形缓冲区在用掉一个数据元素之后，剩下的所有元素都要向前推进，就像排队一样。在使用方面，环形缓冲区更加适合用于实现先进先出缓冲区，而非环形缓冲区用于先进后出缓冲区。

<!--more-->

要理解这种先进先出的缓冲区，可以假设现在有一群人，这一群人的数量用 Size 来表示。通常情况下这个 Size 可以是任意一个数，为了实现方便，这个 Size 现在被限定是 2 的任意次幂。现在这群人要围绕着篝火进行有序的入座，要确定入座的初始位置和最终位置，以及入座的这个人。

```go
type CircularBuffer[T any] struct {
  Size int
  Start int
  End int
  Elements []T
}
```

入座的初始位置使用 Start 表示，最后入座的位置使用 End 表示，入座的所有人使用 Elements 来表示。在进入这个入座的流程之前，我们要对场地进行布置，并指定一些规则。

```go
func (c *CircularBuffer[T]) Initialze(size int) {
  c.Size = size
  c.Start = 0
  c.End = 0
  c.Elements = make([]T, size)
}
```

首先，确定本次入座的所有人数，然后为这些人安排好符合数量的座位，由于现在还没有人入座，因此无法确定哪个座位是初始的，故设置 Start 和 End 都为 0。

```go
func (c *CircularBuffer[T]) IsFull() bool {
  return c.End == (c.Start ^ c.Size)
}
```

这是一种当 Size 为 2 的任意次幂的特殊写法，用于判断此时环形缓冲区是否已经被写满了。用白话总结就是此时的写指针正好追上了读指针，并且绕过一圈了，此时就是缓冲区已满的状态。

```go
func (c *CircularBuffer[T]) IsEmpty() bool {
  return c.End == c.Start
}
```

如果要判断空呢？直接检查写指针和读指针的值是否相同即可。

```go
func (c *CircularBuffer[T]) Increase(ptr int) int {
  return (prt + 1) & (2*c.Size - 1)
}
```

假设目前一共分配了 8 个座位，即 Size = 8。

```
[0] [1] [2] [3] [4] [5] [6] [7]
```

每次进行入座时人们都会有一个动作，就是向下一个座位进行入座。Increase 就是找到下一个座位，如果走到了最后一个座位，就会继续顺时针回到开头继续排列。

```go
func (c *CircularBuffer[T]) Write(e T) {
  c.Elements[c.End&(c.Size-1)] = e
  if c.IsFull() {
    c.Start = c.Increase(c.Start)
  }
  c.End = c.Increase(c.End)
}

func (c *CircularBuffer[T]) Read () T {
  element := c.Elements[c.Start&(c.Size-1)]
  c.Start = c.Increase(c.Start)
  return element
}
```

就这样进行一轮一轮的转动，先进入环形缓冲区的元素就会由于环形缓冲区的转动被读出，也就是所谓的先进先出。

```go
package main

import "fmt"

type CircularBuffer[T any] struct {
 Size     int
 Start    int
 End      int
 Elements []T
}

func (c *CircularBuffer[T]) Initialze(size int) {
 c.Size = size
 c.Start = 0
 c.End = 0
 c.Elements = make([]T, size)
}

func (c *CircularBuffer[T]) Print() {
 fmt.Printf("size = 0x%x, start = %d, end = %d\n", c.Size, c.Start, c.End)
}

func (c *CircularBuffer[T]) IsFull() bool {
 return c.End == (c.Start ^ c.Size)
}

func (c *CircularBuffer[T]) IsEmpty() bool {
 return c.End == c.Start
}

func (c *CircularBuffer[T]) Increase(ptr int) int {
 return (ptr + 1) & (2*c.Size - 1)
}

func (c *CircularBuffer[T]) Write(e T) {
 c.Elements[c.End&(c.Size-1)] = e
 if c.IsFull() {
  c.Start = c.Increase(c.Start)
 }
 c.End = c.Increase(c.End)
}

func (c *CircularBuffer[T]) Read() T {
 element := c.Elements[c.Start&(c.Size-1)]
 c.Start = c.Increase(c.Start)
 return element
}

func main() {
 // Initialze an new CircularBuffer
 var cb CircularBuffer[int]
 cb.Initialze(2)
 cb.Print()

 // Write 1
 cb.Write(1)
 cb.Print()
 fmt.Println(cb.Read())

 // Write 2
 cb.Write(2)
 cb.Print()
}
```

这个案例演示了向一个长度为 2 的环形缓冲区中存入两个值并且读取。

---
{data-content = " 引用 "}

1. [环形缓冲区 - Wikipedia](https://zh.wikipedia.org/wiki/%E7%92%B0%E5%BD%A2%E7%B7%A9%E8%A1%9D%E5%8D%80)
