# 描述

Mutex是为了解决并发过程中的竞争问题而建立的一种并发控制机制，Mutex是一种互斥锁

## 临界区

程序中因并发访问而导致意想不到的结果的代码片段，就叫作临界区。

临界区的代码，在并发访问中，需要被保护起来，同一时间只能有一个进程/线程访问

临界区针对的是一些共享资源，而这些共享资源需要被串行访问

![并发访问](./pics/concurrent_access.jpg)

```golang
package main

import (
	"fmt"
	"sync"
)

func main() {
	count := 0
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100000; j++ {
				count++
			}
		}()
	}
	wg.Wait()
	fmt.Println(count)
}

// 411305
```

在上边的示例中，`count`就是共享资源，在没有被保护的情况下，10个线程对其进行操作，操作的结果明显不是我们预期的结果

**通过 `go race detector` 监控共享变量的同步访问**

执行脚本时，通过`-race`参数，可以一键启动 `go race detector`

```golang
➜  go-mod-test git:(main) ✗ go run -race main.go
==================
WARNING: DATA RACE
Read at 0x00c00013e008 by goroutine 8:
  main.main.func1()
      /Users/xiaosongliu/go/go-mod-test/main.go:16 +0x78

Previous write at 0x00c00013e008 by goroutine 7:
  main.main.func1()
      /Users/xiaosongliu/go/go-mod-test/main.go:16 +0x91

Goroutine 8 (running) created at:
  main.main()
      /Users/xiaosongliu/go/go-mod-test/main.go:13 +0xe4

Goroutine 7 (running) created at:
  main.main()
      /Users/xiaosongliu/go/go-mod-test/main.go:13 +0xe4
==================
==================
WARNING: DATA RACE
Read at 0x00c00013e008 by goroutine 9:
  main.main.func1()
      /Users/xiaosongliu/go/go-mod-test/main.go:16 +0x78

Previous write at 0x00c00013e008 by goroutine 7:
  main.main.func1()
      /Users/xiaosongliu/go/go-mod-test/main.go:16 +0x91

Goroutine 9 (running) created at:
  main.main()
      /Users/xiaosongliu/go/go-mod-test/main.go:13 +0xe4

Goroutine 7 (running) created at:
  main.main()
      /Users/xiaosongliu/go/go-mod-test/main.go:13 +0xe4
==================
322643
Found 2 data race(s)
exit status 66

```

# 基本使用方法

```golang
package main

import (
	"fmt"
	"sync"
)

func main() {
	count := 0
	var wg sync.WaitGroup
	var mu sync.Mutex
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100000; j++ {
				mu.Lock()
				count++
				mu.Unlock()
			}
		}()
	}
	wg.Wait()
	fmt.Println(count)
}

// 1000000
```

嵌入到其他struct中使用

```golang
package main

import (
	"fmt"
	"sync"
)

type Counter struct {
	mu sync.Mutex
	count uint64
}

func main() {
	counter := Counter{}
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100000; j++ {
				counter.mu.Lock()
				counter.count++
				counter.mu.Unlock()
			}
		}()
	}
	wg.Wait()
	fmt.Println(counter.count)
}

// 1000000
```

或是进行一下封装，对外不暴露加锁逻辑

```golang
package main

import (
	"fmt"
	"sync"
)

type Counter struct {
	mu    sync.Mutex
	count uint64
}

func (c *Counter) Incr() {
	c.mu.Lock()
	c.count++
	c.mu.Unlock()
}

func main() {
	counter := Counter{}
	var wg sync.WaitGroup
	wg.Add(10)
	for i := 0; i < 10; i++ {
		go func() {
			defer wg.Done()
			for j := 0; j < 100000; j++ {
				counter.Incr()
			}
		}()
	}
	wg.Wait()
	fmt.Println(counter.count)
}

// 1000000

```

锁的抢占和释放是非常简便的，只要调用 `Lock()` 和 `Unlock()`方法即可

# 实现原理

## 第一版

第一版的`sync.Mutex`,通过一个标识位`key`表示锁是否被持有，

```golang
type Mutex struct {
	key int32;
	sema int32;
}
```

- `key = 0`表示锁还没有被获取
- `key = 1`表示当前goroutine获取到了锁
- `key > 1`表示当前goroutine需要等待锁被释放

**sema:** 是一个信号量变量，用来控制等待goroutine的阻塞休眠和唤醒

**Lock方法实现:**

```golang
func (m *Mutex) Lock() {
	if xadd(&m.key, 1) == 1 {
		// changed from 0 to 1; we hold lock
		return;
	}
	semacquire(&m.sema);
}
```

- 首先是加锁，加锁成功后返回key值，如果`key = 1`，则说明获取到锁，直接返回执行`临界区`代码
- 如果锁被别的 goroutine 获取，则调用`semacquire`方法，使用信号量`sema`将自己休眠，等锁释放的时候，信号量会将它唤醒
  
**Unlock方法实现:**

```golang

func (m *Mutex) Unlock() {
	if xadd(&m.key, -1) == 0 {
		// changed from 1 to 0; no contention
		return;
	}
	semrelease(&m.sema);
}
```

- 首先是释放锁，释放成功后返回key值，如果`key = 0`，说明当前没有等待中的 goroutine，则直接退出即可
- 如果存在等待中的 goroutine ，则通过信号量唤醒 goroutine

加锁和释放锁有个非常重要的方法`xadd`，其实现逻辑如下：

```golang
func xadd(val *int32, delta int32) (new int32) {
	for {
		v := *val;
		if cas(val, v, v+delta) {
			return v+delta;
		}
	}
	panic("unreached")
}
```

其实现原理是：循环调用`cas`方法，对`key`进行加`delta`值操作，直到操作成功，需要说明的是cas方法是一个原子操作

cas指令说明：

`func cas(val *int32, old, new int32) bool`

将给定值`old`与一个内存地址中的值`val`进行比较，如果相等，则用新值`new`替换内存中的值，这个操作是一个原子操作，这个很重要，`sync.Mutex`实现的基础就是这个原子操作，要不然无法保证上锁的原子性，也就无法对`临界区`的代码进行保护

到此，第一版的`sync.Mutex`实现原理就说完了，以下是需要注意的点：

- `sync.Mutex` 与 `goroutine` 没有关联关系，所以 `sync.Mutex` 并不知道谁在加锁，谁在释放锁，也就是说任何一个`goroutine`都可以加锁，也都可以释放锁，最可怕的是还可以只加锁或只释放锁，所以使用时一定要注意，加锁和释放锁要成对出现，要不然`临界区`代码的原子性将不能被保证

第一版代码在`commit：d90e7cbac65c5792ce312ee82fbe03a5dfc98c6f`版本上，有兴趣的可以读一下源码

