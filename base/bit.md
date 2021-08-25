# 位操作

## 输出位

```golang
fmt.Printf("%08b\n", 1)

// 00000001
```

## & 运算

> & 运算符在两个整数间执行`按位AND`操作

左位|右位|结果
:--:|:--:|:--:
1|1|1
1|0|0
0|1|0
0|0|0

```golang
func main() {
	fmt.Printf("%08b\n", 0)
	fmt.Printf("%08b\n", 1)
	fmt.Printf("%08b\n", 0 & 0)
	fmt.Printf("%08b\n", 0 & 1)
	fmt.Printf("%08b\n", 1 & 1)
}

// 00000000
// 00000001
// 00000000
// 00000000
// 00000001
```

## | 运算

> | 运算符在两个整数间执行`按位OR`操作

```golang
func main() {
	fmt.Printf("%08b\n", 0)
	fmt.Printf("%08b\n", 1)
	fmt.Printf("%08b\n", 0 | 0)
	fmt.Printf("%08b\n", 0 | 1)
	fmt.Printf("%08b\n", 1 | 1)
}

// 00000000
// 00000001
// 00000000
// 00000001
// 00000001
```

## ^ 运算

> ^ 运算符在两个整数间执行`按位OR`操作

```golang
func main() {
	fmt.Printf("%08b\n", 0)
	fmt.Printf("%08b\n", 1)
	fmt.Printf("%08b\n", 0 | 0)
	fmt.Printf("%08b\n", 0 | 1)
	fmt.Printf("%08b\n", 1 | 1)
}

// 00000000
// 00000001
// 00000000
// 00000001
// 00000001
```



