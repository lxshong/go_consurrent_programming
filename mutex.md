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
