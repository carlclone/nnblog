# 整数转二进制字符串题目


## 题目

将整数(整数负数和 0) 转换成补码(two's complement) 的二进制表示

```
Input:9
OutPut:1001

Input:0
OutPut:0

Input:-20
OutPut:11111111111111111111111111101100
```

## 解答
- 以前的我的错误写法演示:
```go

func tobinAryString3(n int) string {
	if n ==0 {
		return "0"
	}
	r:=uint32(n)  //go 里没有>>> , 负数得先强转后当正数来操作
	var runeArr string
	m:=make(map[uint32]string)
	m[0]="0"
	m[1]="1"
	for r!=0 {
		runeArr=m[r&1] +runeArr
		r=r>>1
	}
	return runeArr
}
```

问题 1: go 的 string 是不可变类型,每次修改都会重新 allocate memory, 

问题 2: 从前面进行插入,意味着如果是 []byte 或者[]rune数组,每次插入都要执行 O(n)的元素移动,时间复杂度变成O(n^2) , 优化方法是倒序插入, 再做翻转 , 总体复杂度 O(n)


- 改用[]rune ([]byte 也可以) , 优化了空间复杂度
```go

//空间复杂度优化到 O(n) , 时间复杂度 O(n)
func tobinAryString2(n int) string {
	if n ==0 {
		return "0"
	}
	r:=uint32(n)  //go 里没有>>> , 负数得先强转后当正数来操作 
	var runeArr []rune
	m:=make(map[uint32]rune)
	m[0]='0'
	m[1]='1'
	for r!=0 {
		runeArr = append(runeArr,m[r&1])
		r=r>>1
	}

	//原地翻转
	p1,p2:=0,len(runeArr)-1
	for p1<p2 {
		tmp:= runeArr[p1]
		runeArr[p1]= runeArr[p2]
		runeArr[p2]=tmp
		p1++
		p2--
	}
	return string(runeArr)
}



```

- 试用了 strings.Builder , 但翻转的时候使用了额外的空间
```go
//时间复杂度为从(最右一位)到(最左一位1) 的位数 , 假设是 n 位 , 大致为  O(n)
//空间复杂度O(2n)  , builder 为 O(n) , rune[]为 O(n) , 
func tobinAryString(n int) string {
	if n ==0 {
		return "0"
	}
	r:=uint32(n)
	var sb strings.Builder
	for r!=0 {
		sb.WriteString(strconv.Itoa(int(r&1)))
		r=r>>1
	}

	//strings.Builder 的底层 byte 数组是私有的...执行不了原地翻转
	res:=[]rune{}
	for i:=len(sb.String())-1;i>=0;i-- {
		res=append(res,rune(sb.String()[i]))
	}
	return string(res)
}
```
 


## 其他

> 不要记忆逻辑右移和算术右移

- 在 google和 youtube 英文解答中 , 有一个更好的叫法 unsigned bit shift operator , 无符号位移(>>>) , go 里没有,但可以模拟,参考这篇[Equivalent to Java's ">>>" operator in Go?](https://groups.google.com/g/golang-nuts/c/l05WOjmE_xk) 
- signed bit shift operator , 有符号位移, 右移(>>)补上的位值和符号位一致 , 就这么简单
- 左移(<<)的话就没有符号位的区分,都是补 0

> n&1取得最右一位的 bit

> n>>>1 移除最右一位的 bit

> 当 n==0 停止

> [如何获取某个整数类型的最大最小值](https://stackoverflow.com/questions/6878590/the-maximum-value-for-an-int-type-in-go)

```go
const MaxUint = ^uint(0) 
const MinUint = 0 
const MaxInt = int(MaxUint >> 1) 
const MinInt = -MaxInt - 1



uint8  : 0 to 255 
uint16 : 0 to 65535 
uint32 : 0 to 4294967295 
uint64 : 0 to 18446744073709551615 
int8   : -128 to 127 
int16  : -32768 to 32767 
int32  : -2147483648 to 2147483647 
int64  : -9223372036854775808 to 9223372036854775807

func printMinMaxValue() {
    // integer max
    fmt.Printf("max int64 = %+v\n", math.MaxInt64)
    fmt.Printf("max int32 = %+v\n", math.MaxInt32)
    fmt.Printf("max int16 = %+v\n", math.MaxInt16)

    // integer min
    fmt.Printf("min int64 = %+v\n", math.MinInt64)
    fmt.Printf("min int32 = %+v\n", math.MinInt32)

    fmt.Printf("max flloat64= %+v\n", math.MaxFloat64)
    fmt.Printf("max float32= %+v\n", math.MaxFloat32)

    // etc you can see more int the `math`package
}

max int64 = 9223372036854775807
max int32 = 2147483647
max int16 = 32767
min int64 = -9223372036854775808
min int32 = -2147483648
max flloat64= 1.7976931348623157e+308
max float32= 3.4028234663852886e+38
```

 

## 参考资料

更好的实现可以参考 java 的 toBinaryString()方法

[Implementing my own Integer.toBinaryString(int n) method](https://codereview.stackexchange.com/questions/144913/implementing-my-own-integer-tobinarystringint-n-method)

[补码复习](https://www.cnblogs.com/simpleito/p/10787827.html)

[补码复习 2](https://jingyan.baidu.com/article/29697b9106eb52ab21de3c7a.html)

[java源码Integer类toBinaryString（）方法探究](https://blog.csdn.net/u013190513/article/details/70216730/)

[toBinaryString()方法源码解析](https://blog.csdn.net/u014218090/article/details/80200966)

[Bit Shifting](https://www.interviewcake.com/concept/java/bit-shift)


## 近似题目
leetcode 1017题 负二进制