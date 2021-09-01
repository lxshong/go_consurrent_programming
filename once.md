# once

一个简约而不简单的并发原语

## 简介

once可用来执行且仅仅需要执行一次的动作，常常用于单例对象的初始化场景

## 初始化单例方式

package 级别变量初始化，这样在程序启动加载包时，就进行初始化了

```golang
package tools

import (
	"fmt"
	"time"
)

var startTime = time.Now()

func Test() {
	fmt.Println("当前时间：", startTime)
}
```

在init函数中进行初始化

```golang
package tools

import (
	"fmt"
	"time"
)

var startTime time.Time

func init() {
	startTime = time.Now()
}

func Test() {
	fmt.Println("当前时间：", startTime)
}
```

提供显示初始化的方法，在`main`中进行初始化

```golang
package tools

import (
	"fmt"
	"time"
)

var startTime time.Time

func InitTime() {
	startTime = time.Now()
}

func Test() {
	fmt.Println("当前时间：", startTime)
}

// main

package main

import "tools"

// 初始化
func init() {
	tools.InitTime()
}

func main() {
	tools.Test()
}
```

上边三种初始化的方式，都可以很好的对单例进行初始化，但是它们都是在程序初始化时，就对单例进行了初始化，如果我们想要延迟初始化，就需要采用其他方式了

**手写单例** 

```golang
package tools

import (
	"fmt"
	"sync"
	"time"
)

var startTime *time.Time
var mt sync.Mutex

func getTime() *time.Time {
	mt.Lock()
	if startTime == nil {
		fmt.Println("初始化start")
		t := time.Now()
		startTime = &t
		fmt.Println("初始化end")
	}
	mt.Unlock()
	return startTime
}

func Test() {
	wg := sync.WaitGroup{}
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func(i int, wg *sync.WaitGroup) {
			fmt.Println(fmt.Sprintf("%d当前时间：", i), getTime())
			wg.Done()
		}(i, &wg)
	}
	wg.Wait()
}

// 初始化start
// 初始化end
// 0当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
// 1当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
// 6当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
// 4当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
// 2当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
// 7当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
// 9当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
// 5当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
// 3当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
// 8当前时间： 2021-09-01 09:54:53.246065 +0800 CST m=+0.000192708
```

手写单例需要通过加锁控制并发问题，代码量还是相当复杂的，而且还容易出错，非并发的情况，还不能用锁

**once实现**

```golang
package tools

import (
	"fmt"
	"sync"
	"time"
)

var startTime *time.Time
var o sync.Once

func getTime() *time.Time {
	o.Do(func() {
		fmt.Println("初始化start")
		t := time.Now()
		startTime = &t
		fmt.Println("初始化end")
	})
	return startTime
}

func Test() {
	wg := sync.WaitGroup{}
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func(i int, wg *sync.WaitGroup) {
			fmt.Println(fmt.Sprintf("%d当前时间：", i), getTime())
			wg.Done()
		}(i, &wg)
	}
	wg.Wait()
}

// 初始化start
// 初始化end
// 9当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537
// 5当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537
// 3当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537
// 2当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537
// 1当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537
// 4当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537
// 0当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537
// 7当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537
// 6当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537
// 8当前时间： 2021-09-01 10:00:45.487023 +0800 CST m=+0.000156537

```

通过`sync.Once`实现就方便多了，代码也很简洁

不管是并发还是单进程都能很好的支持

```golang
package tools

import (
	"fmt"
	"sync"
	"time"
)

var startTime *time.Time
var o sync.Once

func getTime() *time.Time {
	o.Do(func() {
		fmt.Println("初始化start")
		t := time.Now()
		startTime = &t
		fmt.Println("初始化end")
	})
	return startTime
}

func Test() {
	fmt.Println("当前时间",getTime())
}

// 初始化start
// 初始化end
// 当前时间 2021-09-01 10:02:20.988632 +0800 CST m=+0.000095446
```

所以说，`sync.Once`在懒加载初始化一些单例时，是非常实用的



## 实现原理

```golang

package sync

import (
	"sync/atomic"
)

type Once struct {

	done uint32
	m    Mutex
}

func (o *Once) Do(f func()) {

	if atomic.LoadUint32(&o.done) == 0 {

		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}

```

once 的实现原理比较简单，通过一个标识位`done`判断`f`是否执行过，但是它这里使用了`双检测机制`，这个是为了解决避免所有协程进来后，都先陷入争抢锁的队列中，如果标识位`done == 1` 的话，协程就没必要往下执行了；使用`互斥锁`是为了避免`f`被重复执行；而这里执行完`f`再把`done`置为1，是为了防止，如果`f`方法执行时间比较长，当其他协程判断`done == 1`以后，就直接去调用单例，这时候单例可能还没有初始化完成。

通过这几点的设计，可以看出，底层的设计是很巧妙的

