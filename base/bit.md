# 位操作

## 基础

> 计算机中整数的存储是以补码形式存储的

### 原码

> 对于无符号整数，所有位数都用来表示数值大小

```
1 : 0000 0001
129 : 1000 0001
```

> 对于有符号整数，首位字符表示正负，其余位用来表示数值大小

```
1 : 0000 0001
-1 : 1000 0001
```

### 反码

> 对于无符号整数，反码就是其本身

```
1 : 0000 0001
129 : 1000 0001
```

> 对于有符号整数，正数就是其本身；负数就是在符号位不变的基础上，其他位取反

```
1 : 0000 0001
-1 : 1111 1110
```

### 补码

> 对于无符号整数，补码就是其本身

```
1 : 0000 0001
129 : 1000 0001
```

> 对于有符号整数，正数就是其本身；负数就是在反码的基础上 +1

```
1 : 0000 0001
-1 : 1111 1111
```

## 输出位

```golang
fmt.Printf("%08b\n", 1)

// 00000001
```

## & 运算

> & 运算符在两个整数间执行`按位AND`操作,操作定义如下

```
var a,b
if a == b:
    AND(a,b) = 1
else:
    AND(a,b) = 0
```

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

> | 运算符在两个整数间执行`按位OR`操作,操作定义如下：

```
var a,b
if a == b:
    OR(a,b) = 1
else:
    OR(a,b) = 0
```

左位|右位|结果
:--:|:--:|:--:
1|1|1
1|0|1
0|1|1
0|0|0

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

**异或**

> ^ 运算符在两个整数间执行`按位异或`操作（运算符左右位不相等为1，相等为0），操作定义如下

```
var a,b
if a != b :
    XOR(a,b) = 1
else :
    XOR(a,b) = 0
```


左位|右位|结果
:--:|:--:|:--:
1|1|0
1|0|1
0|1|1
0|0|0

```golang
func main() {
	fmt.Printf("%08b\n", 1^0)
	fmt.Printf("%08b\n", 1^1)
	fmt.Printf("%08b\n", 0^0)
	fmt.Printf("%08b\n", 0^1)
}

// 00000001
// 00000000
// 00000000
// 00000001
```

**取反**

> ^ 运算符还可以用来表示`取反`操作,操作定义如下

```
var a
if a == 1:
    NOT(a) = 0
else:
    NOT(a) = 1
```

右位|结果
:--:|:--:
1|0
0|1

```golang
func main() {
	var a byte = 1
	fmt.Printf("%08b\n", a)
	fmt.Printf("%08b\n", ^a)
	fmt.Printf("%d\n", 1)
	fmt.Printf("%d\n", ^1)
}

// 00000001
// 11111110
// 1
// -2
```

### 1 取反后输出 -2 ？

> 1在计算机中存在为`0000 0001`,取反后就是`1111 1110`,如果以有符号整型输出，则需要进行转码
> `1111 1110` 转码后输出就是-2


## &^ 操作符

> &^ 操作符意为 与非，是 与 和 非 操作符的简写形式，操作定义如下

```
var a,b
AND_NOT(a, b) = AND(a, NOT(b))
```

左位|右位|结果
:--:|:--:|:--:
1|1|0
1|0|1
0|1|0
0|0|0


```golang
func main() {
	fmt.Printf("%08b\n", 1 & ^0)
	fmt.Printf("%08b\n", 1& ^1)
	fmt.Printf("%08b\n", 0& ^0)
	fmt.Printf("%08b\n", 0& ^1)
}

00000001
00000000
00000000
00000000
```

## << 运算符

> << 表示左移运算符,所有位向左移动n位，右侧补0，操作定义为：

```
a = a * 2
```

示例：

```golang
func main() {
	fmt.Printf("%08b\n", 1 << 1)
	fmt.Printf("%08b\n", 1 << 2)
	fmt.Printf("%d\n", 1 << 1)
	fmt.Printf("%d\n", 1 << 2)
}

// 00000010
//00000100
//2
//4
```

## >> 运算符

> >> 表示右移运算符,所有位向右移动n位，左侧补0，操作定义为：

```
a = a / 2
```

示例：

```golang
func main() {
	fmt.Printf("%08b\n", 8 >> 1)
	fmt.Printf("%08b\n", 7 >> 1)
	fmt.Printf("%d\n", 8 >> 1)
	fmt.Printf("%d\n", 7 >> 1)
}

// 00000100
//00000011
//4
//3
```

## 使用案例

**原地交换两个整数的值**

```golang
func main() {
	var a,b = -1,2
	fmt.Println(a,b)
	a = a ^ b
	b = a ^ b
	a = a ^ b
	fmt.Println(a,b)
}

// -1 2
//2 -1
```

**求一个正整数二进制表示中，1的个数**

```golang
func main() {
	var a,sum = 200,0
	fmt.Printf("%08b\n",a)
	for a > 0 {
		sum += a&1
		a >>= 1
	}
	fmt.Println(sum)
}

// 11001000
//3
```
