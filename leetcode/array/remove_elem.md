# Remove Element

## References

- [Leetcode 27](https://leetcode-cn.com/problems/remove-element/)
- [Leetcode 26](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)
- [Leetcode 283](https://leetcode-cn.com/problems/move-zeroes/)
- [Leetcode 844](https://leetcode-cn.com/problems/backspace-string-compare/)
- [Leetcode 977](https://leetcode-cn.com/problems/squares-of-a-sorted-array/)


## Two pointer

[Leetcode 27](https://leetcode-cn.com/problems/remove-element/),
[Leetcode 26](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/),
[Leetcode 283](https://leetcode-cn.com/problems/move-zeroes/),
[Leetcode 977](https://leetcode-cn.com/problems/squares-of-a-sorted-array/)
can be resolved by `Two Pointer`

`Two Pointer` has a lot of transformations

1. `Fast` and `Slow` pointers both start at the same side (like [Leetcode 27](https://leetcode-cn.com/problems/remove-element/),[Leetcode 26](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/),[Leetcode 283](https://leetcode-cn.com/problems/move-zeroes/),)
2. Pointers start at different location ([Leetcode 977](https://leetcode-cn.com/problems/squares-of-a-sorted-array/))

It is often used with `sorted array`

### Fast and Slow

Fast and Slow pointers usually do things like:

```
if (some condition) is true:
	only move the fast pointer
else:
	move two pointers
```

The `fast` pointer is like a probe, it testify something that we don't want to involve, and move forward

### Leetcode 209

```go
func minSubArrayLen(target int, nums []int) int {
    notFound := 1000000
    ans := notFound
    for i:=0;i<len(nums);i++{
        var sum int
        j := i
        for ;j<len(nums);j++{
            sum += nums[j]
            if sum >= target {
                ans = min(ans, j-i+1)
                break
            }
        }

        if j == len(nums) {
            break
        }
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

With `Two Pointers`

```go
func minSubArrayLen(target int, nums []int) int {
    notFound := 1000000
    left, right := 0, 0
    // sum included value from [left, right]
    var sum int
    ans := notFound
    for right < len(nums) {
        sum += nums[right]

        // try to move left if possible
        for sum >= target {
            if sum - nums[left] >= target {
                sum -= nums[left]
                left++
            } else {
                break
            } 
        }

        // minimum distance, calculate it
        if sum >= target {
            ans = min(ans, right - left + 1)
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
