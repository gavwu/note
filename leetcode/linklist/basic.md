
## References

- [Leetcode 203](https://leetcode-cn.com/problems/remove-linked-list-elements/)
- [Leetcode 707](https://leetcode-cn.com/problems/design-linked-list/)
- [Leetcode 206](https://leetcode-cn.com/problems/reverse-linked-list/)
- [Leetcode 24](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)
- [Leetcode 19](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)
- [Leetcode 02.07](https://leetcode-cn.com/problems/intersection-of-two-linked-lists-lcci/)
- [Leetcode 142](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

## Use dummy node

The first time I resolve [Leetcode 203](https://leetcode-cn.com/problems/remove-linked-list-elements/), the code is:

```go
func removeElements(head *ListNode, val int) *ListNode {
    if head == nil {
        return nil
    }
    
    tmp := &ListNode{Next: head}

    for head.Next != nil {
        if head.Next.Val == val {
            head.Next = head.Next.Next
        } else {
            head = head.Next
        }
    }

    // handle the first node
    if tmp.Next.Val == val {
        return tmp.Next.Next
    }

    return tmp.Next
}
```

I handle the `first node` specially by checking the value at the end of the code

After a period of time, I found a trick about `dummy` node. Using `dummy` node make our code look more natural:

```go
// no more special handle
func removeElements(head *ListNode, val int) *ListNode {
    dummy := &ListNode{Next: head}

    cur := dummy
    for cur.Next != nil {
        if cur.Next.Val == val {
            cur.Next = cur.Next.Next
        } else {
            cur = cur.Next
        }
    }

    return dummy.Next
}```

## Minimum step

When I was dealing with `changing` the linklist, I found it is very useful to solve the problem by repeating the `minimum step`

> minimum step is the core of changing the linklist, it only contains the necessary node to finish the step

[Leetcode 206](https://leetcode-cn.com/problems/reverse-linked-list/) reverse the linklist

the minimum step would contain only 3 nodes:

- prev: the node before cur
- cur: the node we are operating on
- next: the node after cur

TODO: graft
```plantuml

```

what we do with the whole linklist is just repeat the step, and re-assign those nodes (move the node point to the next node we want to operate)

```go
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    dummy := &ListNode{Next: head}
    cur := dummy.Next

    for cur != nil {
        next := cur.Next
        cur.Next = prev

        prev = cur
        cur = next
    }

    return prev
}
```

[Leetcode 24](https://leetcode-cn.com/problems/swap-nodes-in-pairs/) swap pairs can also use `minimum step`

```go
func swapPairs(head *ListNode) *ListNode {
    res := &ListNode{Next:head}
    prev := res

    for prev != nil {
        cur := prev.Next
        if cur == nil {
            break
        }
        next := cur.Next
        if next == nil {
            break
        }

        prev.Next = next
        cur.Next = next.Next
        next.Next = cur

        prev = cur
    }

    return res.Next
}
```

we can also do that in recursive

```go
func swapPairs(head *ListNode) *ListNode {
    prev := head
    if prev == nil {
        return prev
    }

    cur := prev.Next
    if cur == nil {
        return prev
    }

    next := cur.Next

    cur.Next = prev
    prev.Next = swapPairs(next)

    return cur
}
```

each `swapPairs` do a minimum step and recursively call the next


