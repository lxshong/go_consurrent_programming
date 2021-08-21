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


