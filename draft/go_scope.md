# Go元编程系列--Scope

Scope是许多语言都有的一个概念，关于Scope在编程语言的意义，可以参考一下编译原理相关知识，在这里做一个简单的定义

Scope是一个作用域，就犹如每个变量都有其作用域

```go
// global is in package scope
var global int

func main() {
    var a int
    // a is in scope main
}

func foo() {
    var b int
    // b is in scope foo
}
```

我们从上面例子可以知道

- main函数可以访问`global`和`a`
- foo可以访问`global`和`a`
- foo无法访问main函数的`a`，main函数无法访问foo中定义的`b`

在简单了解了作用域的概念后，我们进入Go的Scope

## 组成

```go
// A Scope maintains a set of objects and links to its containing
// (parent) and contained (children) scopes. Objects may be inserted
// and looked up by name. The zero value for Scope is a ready-to-use
// empty scope.
type Scope struct {
	parent   *Scope
	children []*Scope
	elems    map[string]Object // lazily allocated
	pos, end token.Pos         // scope extent; may be invalid
	...
}
```

从起定义来看，其实Scope是一个`树形`结构，并且在一个Scope里，有我们定义的Object。

还有两个变量记录这个Scope的边界，这里的边界用代码来展示就是`{...}`

整体结构比较简单易懂

### Hello World例子

```go
package main

import "fmt"

func main() {
	const message = "hello, world"
	fmt.Println(message)
}
```

以这个程序为例，总共有4个scope，从上往下

- universe scope -> 保留的，每一个package scope的parent都是universe，但是universe没有children
- package scope -> main(*Func)
- file scope -> fmt(*PkgName)
- function scope(main) -> message ...


如果程序复杂，那么scope也会越多

## Identifier Resolve

我们知道，作用域一大功能就是将变量的Access范围变得明确，就犹如一开始的例子

如何知道在一个Scope里，也就是一个代码块里，能不能使用某一个变量？

从下往上找，如果能找到，说明可以使用，如果不能找到，说明还没声明过。

## 有用的Function

这里面已经封装了很多好用的函数，帮助我们从中获取信息

- `Names() []string` 将scope里的所有elem的name排序后返回
- `Lookup(name string) Object` 可以根据其名字，寻找一个Object
- `LookupParent(name string, pos token.Pos) (*Scope, Object)` 这是一个往上寻找
- `Contains(pos token.Pos) bool` 指明一个position是否在这个Scope中


## 小结一下

Scope提供了非常强大的功能，帮组我们在元编程的时候，寻找我们需要Object

