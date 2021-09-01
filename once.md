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





