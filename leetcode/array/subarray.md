# SubArray

## References

- [Leetcode 209](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)
- [Leetcode 904](https://leetcode-cn.com/problems/fruit-into-baskets/)
- [Leetcode 76](https://leetcode-cn.com/problems/minimum-window-substring/)

## Slice Window

When dealing with `subarray` problems, try `slice window`.

be aware of the boundary (is it included or excluded?)

### Leetcode 209

```go
func minSubArrayLen(target int, nums []int) int {
    notFound := 1000000
    left, right := 0, 0
    var sum int
    ans := notFound
    for right < len(nums) {
        sum += nums[right]

        // try to move left if possible
        for sum >= target {
	// the slice window meet the condition, try do the calculation
            ans = min(ans, right - left + 1)
            sum -= nums[left]
            left++
        }

        // move right
        right++
    }

    if ans == notFound {
        return 0
    }
    return ans
}

func min(a,b int) int {
    if a < b {
        return a
    }
    return b
}
```

### Leetcode 904

```go
func totalFruit(fruits []int) int {
    var f0,f1 int
    var left, right int
    var ans int

    // initailize f0,f1
    f0 = fruits[0]
    for i:=1;i<len(fruits);i++{
        if fruits[i] != f0 {
            f1 = fruits[i]
            right = i
            break
        }
    }
    if f1 == f0 {
        return len(fruits)
    }
    ans = right - left + 1

    right++
    for right < len(fruits) {
        if fruits[right] != f0 && fruits[right] != f1 {
            // hit different fruit, need to replace f0
            sample := fruits[right-1]
            for i:=right-2;i>=0;i--{
                if fruits[i] != sample {
                    left = i + 1
                    break
                }
            }
            f0 = fruits[left]
            f1 = fruits[right]
        }

        ans = max(ans, right-left+1)
        right++
    }

    return ans
}

func max(a,b int) int {
    if a > b {
        return a
    }
    return b
}
```

### Leetcode 76

```go
func minWindow(s string, t string) string {
    if len(s) < len(t) {
        return ""
    }

    var left, right int
    var L, R int
    var notFound = 1000000
    min := notFound

    // record how many characters we need
    needChar := make(map[byte]int)
    for i:=0;i<len(t);i++{
        needChar[t[i]] = needChar[t[i]] + 1
    }

    // check if satisfy all characters
    isSatisfy := func() bool {
        for _,v := range needChar {
            if v > 0 {
                return false
            }
        }

        return true
    }

    for ;right < len(s); right++ {
        c := s[right]
        v, ok := needChar[c]
        if !ok {
            // useless character, make right forward
            continue
        }

        needChar[c] = v - 1

        for left <= right && isSatisfy() {
            // current [left, right] satisfy the condition, try to process the minimum
            // find the most close one from left
            for {
                lc := s[left]
                left++
                if v, ok := needChar[lc]; ok {
                    needChar[lc] = v + 1
                    break
                } 
            }

            // calculate the minimum
            if min > right - left + 1 {
                min = right - left + 1
                L = left - 1
                R = right
                fmt.Printf("left: %d, right: %d\n", L, R)
            }
        }
    }

    if min == notFound {
        return ""
    }

    return s[L:R+1]
}
```
