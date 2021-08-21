# 常量

所谓常量，也就是在程序编译阶段就确定的值，在程序运行过程中无法改变的值

## 定义常量

### 单个定义

```golang
const cst = 1
```

### 批量定义

```golang
const (
	cst1 = 1
	cst2 = 2
	cst3 = 3
	cst4 = 4
	cst5 = 5
	cst6 = 6
	cst7 = 7
)
```

### 常量中包含常量

```golang
const cst_1 = cst + "1"
const cst = "hi"

fmt.Println(cst,cst_1)
// hi hi1
```

### 常量简写定义

```golang
const (
	cst1 = "hi"
	cst2
	cst3
	cst4
	cst5
	cst6
	cst7
)

fmt.Println(cst1,cst2,cst3,cst4,cst5,cst6,cst7)

// hi hi hi hi hi hi hi
```

### 常量中使用iota

- iota适用于批量定义常量
- iota只能在常量的表达式中使用
- iota在const关键字出现时将被重置为0（可以理解为const中常量的下标）

```golang
const (
	cst1 = iota
	cst2
	cst3
	cst4
	cst5
	cst6
	cst7
)

fmt.Println(cst1,cst2,cst3,cst4,cst5,cst6,cst7)
// 0 1 2 3 4 5 6
```

iota 也可以应用于表达式中

```golang
const (
	cst1 = 1 << iota
	cst2
	cst3
	cst4
	cst5
	cst6
	cst7
)

fmt.Println(cst1,cst2,cst3,cst4,cst5,cst6,cst7)
// 1 2 4 8 16 32 64
```

```golang
const (
	cst1 = 1 + iota
	cst2
	cst3
	cst4
	cst5
	cst6
	cst7
)

fmt.Println(cst1,cst2,cst3,cst4,cst5,cst6,cst7)
// 1 2 3 4 5 6 7
```

中间指定变量不使用规律

```golang
const (
	cst1 = 1 + iota
	cst2
	cst3 = 12
	cst4 = 1 + iota
	cst5
	cst6
	cst7
)

fmt.Println(cst1,cst2,cst3,cst4,cst5,cst6,cst7)
// 1 2 12 4 5 6 7
```