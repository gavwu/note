# Binary Search

## References

- [Leetcode 704](https://leetcode-cn.com/problems/binary-search/)
- [Leetcode 35](https://leetcode-cn.com/problems/search-insert-position/)
- [Leetcode 34](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)
- [Leetcode 69](https://leetcode-cn.com/problems/sqrtx/)
- [Leetcode 367](https://leetcode-cn.com/problems/valid-perfect-square/)

## Rules

### Condition

what condition should the `result` satisfy

for [Leetcode 704](https://leetcode-cn.com/problems/binary-search/)

`nums[result] == target`

for [Leetcode 35](https://leetcode-cn.com/problems/search-insert-position/)

`nums[result-1] < target <= nums[result]`

```go
func searchInsert(nums []int, target int) int {
    left, right := 0, len(nums)-1
    ans := len(nums)
    for left <= right{
        mid := (left+right) / 2
        if nums[mid] >= target {
            ans = mid
            right = mid - 1
        } else {
            left = mid + 1
        }
    }

    return ans
}
```

for [Leetcode 34](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

`nums[x] == target && result[0] <= x <= result[1]`

for [Leetcode 69](https://leetcode-cn.com/problems/sqrtx/)

`result * result <= x && (result+1) * (result+1) > x`

for [Leetcode 367](https://leetcode-cn.com/problems/valid-perfect-square/)

`result * result == x`

> IMPORTANT: Make Sure You Know What Value You Should Return

### Boundary

When doing binary search, be aware of boundary

The code really depends on **how you define your boundary** (is it included or excluded)

For example, let's look at [Leetcode 704](https://leetcode-cn.com/problems/binary-search/)

```go
func search(nums []int, target int) int {
    left, right := 0, len(nums)-1
    for left <= right {
        mid := left + (right - left) / 2
        if nums[mid] == target {
            return mid
        }

        if nums[mid] > target {
            right = mid-1
        } else {
            left = mid+1
        }
    }
    return -1
}
```

I defined the `right` as included, so the condition of continue the loop is `left <= right`, not `left < right`

If I defined the `right` to be excluded, the code should be:

```go
func search(nums []int, target int) int {
    left, right := 0, len(nums)
    for left < right {
        mid := left + (right - left) / 2
        if nums[mid] == target {
            return mid
        }

        if nums[mid] > target {
            right = mid
        } else {
            left = mid+1
        }
    }
    return -1
}
```

Two things are changed:

1. the for loop condition is changed from `left <= right` to `left < right`
2. the `right` assignment is changed from `right = mid - 1` to `right = mid`

---

Another good example is [Leetcode 69](https://leetcode-cn.com/problems/sqrtx/)

I was really confusing about whether I should return the `right` or `left` until I clearly defined them

```go
func mySqrt(x int) int {
    if x <= 1 {
        return x
    }
    // left * left <= x
    // right * right > x
    left, right := 0, x

    for left + 1 < right {
        mid := (left+right) / 2

        if x / mid < mid {
            // mid * mid > x
            right = mid
        } else {
            // mid * mid <= x
            left = mid
        }
    }

    return left
}
```

I defined the `right` to be `right * right > x`, and `left` to be `left * left <= x`

the result is coming out of my head right away, and it is so clear that I should return the `left`

and also clear about how should I assign them

## Tips

### How to calculate the middle

I was wondering why people always use `left + (right - left) / 2` instead of `(left + right) / 2`.

The answer is: `(left + right) / 2` might incidently overflow(assume `left` and `right` are huge integer), 
but the `left + (right - left) / 2` won't have this issue

So if you want to calculate the middle of two value, be careful with the overflow

