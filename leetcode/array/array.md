双指针
前缀和
差分数组
花式遍历（螺旋遍历/二维数组旋转）
滑动窗口
二分查找
二分查找的变种
田忌赛马
数组常数时间查找（把无用的数字挪到后面/或者映射）
数组去重

数组

### 二分查找

#### 边界

二分查找中，边界的处理是极为重要的，有时也是非常困扰的

1. 到底是取mid还是mid+1
2. 到底是left<right还是left<=right
3. 到底是返回left还是right，还是left-1等等


比如说Leetcode-69: x的平方根，代码如下

```go
func mySqrt(x int) int {
	// 把left初始化为1，是为了防止下面mid出现0的情况
    left, right := 1, x
	// 定义左闭右闭区间[left,right]
    for left <= right {
        mid := left + (right-left)/2
        tmp := x / mid

        if tmp > mid {
			// 这里不能left = mid，否则可能会陷入无限循环
			// 虽然mid有可能是我们期望的值，但是最终它还是会被right指向
            left = mid + 1
        } else if tmp < mid {
			// 因为是[left,right]，因此right是包含在内的
			// 而这里的mid * mid > x，因此肯定不是我们期望的值
			// 所以 right = mid - 1
            right = mid - 1
        } else {
            return mid
        }
    }

	// 到这里，满足以下公式：（只讨论x > 1情况下）
	// 1. left*left > x
	// 2. right*right < x
	// 按照题目意思，应该是取right
	// 为什么会满足这两个公式呢？
	// 因为到了最后一次循环里，left = right
	// A. (left-1)*(left-1) < x
	// B. (right+1)*(right+1) > x
	// 按照判断条件，要么left = mid + 1，要么right = right - 1
	// 因为left = right，因此left = right = mid
	// 因此转换为，要么left = right + 1，要么right = left - 1
	// 按照A，B条件可以证明1，2

	// 针对x <= 1的情况进行特例分析
	// 当x = 0时right = 0
	// 当x = 1时right = 1
	// 满足条件
    return right
}
```

> Q：到底是取mid还是mid+1

这个与我们用左闭右闭的区间还是其它方式的区间有关，一般来说左闭右闭的区间，不会直接用mid，而是会对齐进行+/-1操作

> Q：到底是left<right还是left<=right

这个也是和区间有关系，一般来说，左闭右闭区间会用<=，左闭右开区间会用<

> Q：到底是返回left还是right，还是其它值

这个是需要去推敲的，首先是确定当循环执行完了之后，left和right分别满足什么条件，题目要求我们返回的值满足什么条件，一般来说，这样对比以下就能够选出正确的值

#### 例题

##### Leetcode 367：有效的完全平方数

```go
func isPerfectSquare(num int) bool {
    left, right := 1, num
    for left <= right {
        mid := left + (right-left)/2
        tmp := num / mid
        if mid == tmp {
            return num % mid == 0
        } else if mid > tmp {
            right = mid - 1
        } else {
            left = mid + 1
        }
    }

    return false
}
```

完全平方数满足：x * x = num

因此我们只需要关注 num / mid == mid的情况，如果mid不满足这个，则可以继续移动“指针”进行下一步计算

##### Leetcode 704: 二分查找

```go
func search(nums []int, target int) int {
    left, right := 0, len(nums)-1
    for left <= right {
        mid := left + (right-left)/2
        if nums[mid] == target {
            return mid
        } else if nums[mid] > target {
            right = mid - 1
        } else {
            left = mid + 1
        }
    }

    return -1
}
```

最简单的二分查找

##### Leetcode 35: 搜索插入位置

```go
func searchInsert(nums []int, target int) int {
    left, right := 0, len(nums)-1
    for left <= right {
        mid := left + (right-left)/2
        if nums[mid] < target {
            left = mid + 1
        } else if nums[mid] > target{
            right = mid - 1
        } else {
			// 可以提前退出，因为没有重复元素
            return mid
        }
    }

	// 到这里满足 
	// left * left > target
	// right * right < target
    return left
}
```

按照题目意思，返回的值x需要满足：x * x >= target

如果拓宽以下题目，升序数组`nums`是有重复元素的，应该怎么解答

如果是有重复元素，我们应该把元素插入在`一号位`上，比如

```
[1,3,3,3,4]
2
[1,3,3,3,4]
3
```

应该获得

```
[1,2,3,3,3,4]
[1,3,3,3,3,4]
   ^
   新插入
```

返回的值x需要满足：x * x >= target

```go
func searchInsert(nums []int, target int) int {
    left, right := 0, len(nums)-1
    for left <= right {
        mid := left + (right-left)/2
        if nums[mid] < target {
            left = mid + 1
        } else if nums[mid] > target{
            right = mid - 1
        } else {
			// 不能提前返回，需要找到左边界
			right = mid - 1
        }
    }

	// 到这里满足 
	// left * left >= target
	// right * right < target
    return left
}
```

按照条件，我们应该返回`left`

### 指针法

#### 快慢指针

快慢指针一般针对解决以下问题：

- 删除特定元素
- 链表找环
- 链表长度差

##### Leetcode 27: 移除元素

```go
func removeElement(nums []int, val int) int {
    fast, slow := 0, 0
    for fast < len(nums) {
        if nums[fast] == val {
            fast++ 
            continue
        }

        nums[slow] = nums[fast]
        slow++
        fast++
    }

    return slow
}
```

思路很简单，快指针负责将不需要的元素忽略，慢指针负责原地填充出新的数组

##### Leetcode 26: 删除有序数组中的重复项

```go
func removeDuplicates(nums []int) int {
    slow, fast := 1,1
    for fast < len(nums) {
        if nums[slow-1] == nums[fast] {
            fast++
            continue
        }

        nums[slow] = nums[fast]
        slow++
        fast++
    }

    return slow
}
```

这一题和27题大同小异，无非是换了一个判断条件

##### Leetcode 283: 移动零

```go
func moveZeroes(nums []int)  {
    slow, fast := 0, 0
    for fast < len(nums) {
        if nums[fast] == 0 {
            fast++
            continue
        }

        nums[slow] = nums[fast]
        slow++
        fast++
    }

    for i:=slow;i<len(nums);i++{
        nums[i] = 0
    }
}
```

也是大同小异，换了个判断条件

#### 双指针比较

##### Leetcode 844: 比较含退格的字符串

刚看到这题的时候，很容易想到重构整个字符串进行比较

```go
func backspaceCompare(s string, t string) bool {
    return formatString(s) == formatString(t)
}

func formatString(s string) string {
    var backspace int
    var res []byte
    for i:=len(s)-1;i>=0;i-- {
        b := s[i]

        if b == '#' {
            backspace++
            continue
        }

        if backspace != 0 {
            backspace--
            continue
        }

        res = append(res, b)
    }

    return string(res)
}
```

题目进阶要求使用O(1)的空间复杂度，说明不能够构造字符串进行对比了

字符串相等，说明它们：

- 长度相等
- 每个字符都相等

既然无法构造出字符串，那么可以按照字符来比较

因为有退格存在，因此我们需要从后往前，从后往前可以确定哪些字符是需要比较的（最终被保留下来的字符）

我们有两个指针，分别指向两个字符串**需要比较的字符**

```go
func backspaceCompare(s string, t string) bool {
	// 初始化为最后的字符
    sp, tp := len(s)-1, len(t)-1
    for {
		// 需要找到下一个有效字符
        sp = nextValid(s, sp)
        tp = nextValid(t, tp)

		// 如果两个字符串都已经遍历完了，说明相等
        if sp == -1 && tp == -1 {
            return true
        }

		// 如果只有其中一个字符串遍历完了，说明不相等
        if sp == -1 || tp == -1 {
            return false
        }

		// 如果字符不相等，那么字符串也不会相等
        if s[sp] != t[tp] {
            return false
        }

		// 指向下一个
        sp--
        tp--
    }
}

// 找出下一个有效字符
func nextValid(s string, pointer int) int {
    skip := 0
    for skip >= 0 && pointer >= 0{
        if s[pointer] == '#' {
            skip++
            pointer--
            continue
        }

        if skip > 0 {
            pointer--
            skip--
            continue
        }

        skip--
    }

    return pointer
}

```

#### 前后指针

##### Leetcode 977: 有序数组的平方

此题一开始我还在找“中心点”，然后沿着这个“中心点”向两边散开，取平方最小的加入结果集

其实这题用前后指针，按照从大到小进行插入即可，非常简单

```go
func sortedSquares(nums []int) []int {
    left, right := 0, len(nums)-1
    res := make([]int, len(nums))
    idx := len(res)-1
    for left <= right {
        v1, v2 := pow2(nums[left]), pow2(nums[right])
        if v1 > v2 {
            res[idx] = v1
            left++
        } else {
            res[idx] = v2
            right--
        }
        idx--
    }

    return res
}

func pow2(x int) int {
    return x*x
}
```

#### 滑动窗口

[无敌代码框架](https://labuladong.github.io/algo/2/18/25/)

用这个框架解决了904，76，209

##### Leetcode 904: 水果成篮

```go
func totalFruit(fruits []int) int {
    var res int
    count := make(map[int]int)
	// [left,right) 区间
    left, right := 0, 0
    for right < len(fruits) {
        f := fruits[right]
		// 这里right++是为了满足[left,right)的区间定义
        right++

		// 做一些计算/统计
        count[f]++

		// 当满足条件时，尝试找到边界值
		// 并且再次破坏其满足条件才能进入下一个循环

		// 最多只能有两种水果
		// 如果超过了，需要循环减去之前的种类，直到满足该条件
        for len(count) > 2 {
            removed := fruits[left]
            count[removed]--
            if count[removed] == 0 {
                delete(count, removed)
            }
            left++
        }

		// 计算该结果，需要保证到这里，是满足水果种类只有2种
        res = max(res, right-left)
    }

    return res
}

func max(i,j int) int {
    if i > j {
        return i
    }
    return j
}
```

在[left, right)的滑动窗口里，满足水果的种类小于等于2，我们需要做的就是如何保持这个约定

在超过条件时，应该收缩左边界，使其最终又满足条件

上面代码的计数方式还可以优化一下，用数组操作会更高效

```go
func totalFruit(fruits []int) int {
    var res int
    count := make([]int, len(fruits))
    var totalType int
    left, right := 0, 0
    for right < len(fruits) {
        f := fruits[right]
        right++

        count[f]++
        if count[f] == 1 {
            totalType++
        }

        for totalType > 2 {
            removed := fruits[left]
            count[removed]--
            if count[removed] == 0 {
                totalType--
            }
            left++
        }

        res = max(res, right-left)
    }

    return res
}

func max(i,j int) int {
    if i > j {
        return i
    }
    return j
}
```

##### Leetcode 76: 最小覆盖字串

```go
func minWindow(s string, t string) string {
    var res string
    need := make(map[byte]int)
    var needCount int
    for i:=0;i<len(t);i++{
        need[t[i]]++
        if need[t[i]] == 1 {
            needCount++
        }
    }
    var valid int
    window := make(map[byte]int)

    var left, right int
    for right < len(s) {
        b := s[right]
        right++

		// 统计/计算
        if need[b] > 0 {
            window[b]++
            if window[b] == need[b] {
                valid++
            }
        }

		// 满足条件时，计算结果，并且尝试让其再次不满足条件（通过收缩窗口）
        for valid == needCount {
            if res == "" || right - left < len(res) {
                res = s[left:right]
            }
            removed := s[left]
            if need[removed] > 0 {
                if window[removed] == need[removed] {
                    valid--
                }
                window[removed]--
            }
            left++
        }
    }

    return res
}
```

优化成数组, 获取数组的速度比map要快

```go
func minWindow(s string, t string) string {
    var res string
    need := make([]int, 256)
    var needCount int
    for i:=0;i<len(t);i++{
        idx := getIndex(t[i])
        need[idx]++
        if need[idx] == 1 {
            needCount++
        }
    }
    var valid int
    window := make([]int, 256)

    var left, right int
    for right < len(s) {
        b := s[right]
        idx := getIndex(b)
        right++

        if need[idx] > 0 {
            window[idx]++
            if window[idx] == need[idx] {
                valid++
            }
        }

        for valid == needCount {
            if res == "" || right - left < len(res) {
                res = s[left:right]
            }
            removed := getIndex(s[left])
            if need[removed] > 0 {
                if window[removed] == need[removed] {
                    valid--
                }
                window[removed]--
            }
            left++
        }
    }

    return res
}

func getIndex(b byte) int {
    return int(b - 'a')
}
```

##### Leetcode 209: 长度最小的子数组

类似上面，这一题还相对简单

```go
func minSubArrayLen(target int, nums []int) int {
    var sum int
    var res int
    var left, right int
    for right < len(nums) {
        num := nums[right]
        right++

        sum += num

        for sum >= target {
            if res == 0 || right - left < res {
                res = right - left
            }

            sum -= nums[left]
            left++
        }
    }

    return res
}
```

### 花式遍历（矩阵）

### 前缀和

前缀和常用于减少运算次数（相当于把中间数据提前加工好，后面方便使用）

##### Leetcode 303: 区域和检索 - 数组不可变

```go
type NumArray struct {
    preSum []int
}


func Constructor(nums []int) NumArray {
    preSum := make([]int, len(nums)+1)
    for i:=1;i<len(preSum);i++{
        preSum[i] = preSum[i-1] + nums[i-1]
    }

    return NumArray{preSum:preSum}
}


func (this *NumArray) SumRange(left int, right int) int {
    return this.preSum[right+1] - this.preSum[left]
}
```

前缀和其实也是为了快速计算，SumRange是O(1)的复杂度

这一题从实现上完全可以用for循环实现

```go
func (this *NumArray) SumRange(left int, right int) int {
		var sum int
		for i:=left;i<=right;i++{
			sum += this.nums[i]
		}

		return sum
}
```

SumRange是O(n)的复杂度

##### Leetcode 304: 二维区域和检索 - 矩阵不可变

这一题是303的二维版本，本质上没有变，只是计算上变复杂了，边界上需要更小心

```go
type NumMatrix struct {
    preSum [][]int
}

func Constructor(matrix [][]int) NumMatrix {
    m := len(matrix)
    n := len(matrix[0])

	// 注意边界
    preSum := make([][]int, m+1)
    for i:=0;i<m+1;i++{
        preSum[i] = make([]int, n+1)
    }

	// 注意边界
    for row:=1;row<m+1;row++{
        for col:=1;col<n+1;col++{
            preSum[row][col] = preSum[row][col-1] + preSum[row-1][col] - preSum[row-1][col-1] + matrix[row-1][col-1]
        }
    }

    return NumMatrix{preSum:preSum}
}


func (this *NumMatrix) SumRegion(row1 int, col1 int, row2 int, col2 int) int {
    return this.preSum[row2+1][col2+1] - this.preSum[row1][col2+1] - this.preSum[row2+1][col1] + this.preSum[row1][col1]
}
```

二维的画个图可能会更好理解

### 差分数组

差分数组解决的问题和**前缀和**很类似，都是为了解决大量的计算问题（降低时间复杂度）

特征：

- 多个区间的值加减

##### Leetcode 1109: 航班预定统计

```go
func corpFlightBookings(bookings [][]int, n int) []int {
    diff := make([]int, n)
    for _, booking := range bookings {
        start, end := booking[0], booking[1]
        val := booking[2]

		// 因为数组下标是从0开始，因此这里-1
		// 加和的范围是[start, end]闭区间
        diff[start-1] += val
		// 如果end >= n，没必要去减了
        if end < n {
            diff[end] -= val
        }
    }

	// 根据差分数组得到答案
    res := make([]int, n)
    res[0] = diff[0]
    for i:=1;i<n;i++{
        res[i] = res[i-1]+diff[i]
    }

    return res
}
```

##### Leetcode 1094: 拼车

```go
func carPooling(trips [][]int, capacity int) bool {
    diff := make([]int, 1000)
    for _, trip := range trips {
        passengers := trip[0]
        from, to := trip[1], trip[2]

        diff[from] += passengers
        if to < 1000 {
            diff[to] -= passengers
        }
    }

    var sum int
    for i:=0;i<1000;i++{
        sum += diff[i]
        if sum > capacity {
            return false
        }
    }

    return true
}
```
