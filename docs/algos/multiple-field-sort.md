# 使用快排思想实现多字段排序


思想: 每次 partition 都把其中一个元素放到排序好后正确的位置

## 问题
按第一个字段升序 , 第二个字段降序

## 例子
```
func main() {
	s := [][]int{
		{2, 1},
		{1, 1},
		{3, 2},
		{1, 3},
		{3, 1},
		{1, 2},
	}
	fmt.Println(sortArray(s))
}

-> [[1 3] [1 2] [1 1] [2 1] [3 2] [3 1]]
```

## 题解


只需要在快排的partition操作中多加一层判断,把元素放到正确的位置

从排序函数中抽离出比较函数,comparator

partition操作代码
```
func partition2(nums [][]int, l int, r int) int {
	var (
		j, i int
	)
	// [l+1,j] <v , [j+1,i)>v

	v := nums[l]
	j = l
	for i = l + 1; i <= r; i++ {
        //根据第二个字段降序
		if nums[i][0] == v[0] {
			if nums[i][1] > v[1] {
				swap(nums, i, j+1)
				j++
			}

		} else if nums[i][0] < v[0] {
			swap(nums, i, j+1)
			j++
		}

	}
	swap(nums, l, j)
	return j
}
```

[完整代码](https://github.com/carlclone/Algorithms-in-Go/blob/master/sort/sort-mutliple-array.go)

## 后续参考改进

[快排实现仿order by多字段排序](https://www.cnblogs.com/zengchunyun/p/10384372.html)

[thenBy.j](https://github.com/Teun/thenBy.j)