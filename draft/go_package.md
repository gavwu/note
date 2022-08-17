# Go元编程系列--Package

## pre-requisite

- [Scope](go_scope.md)

package一般是我们做代码分析的入口

(此处可以放张图解释一下整体的结构)

## 定义

```go
// A Package describes a Go package.
type Package struct {
	path     string
	name     string
	scope    *Scope
	complete bool
	imports  []*Package
}
```

## Scope

`scope`为整个package的`scope`，比如说一个程序

```go
package main

import "fmt"

type People struct {
	age int
	name string
}

func main() {
        fmt.Println("Hello, world")
}
```

所对应的package.scope是这样的

```
(dlv) p pkg.scope
*go/types.Scope {
	parent: *go/types.Scope {
        ...
    },
	children: []*go/types.Scope len: 1, cap: 1, [
		*(*"go/types.Scope")(0xc0000e14a0),
	],
	elems: map[string]go/types.Object [
		"People": ...,
		"main": ...,
	],
	pos: NoPos (0),
	end: NoPos (0),
	comment: "package \"cmd/hello\"",
	isFunc: false,}
```

我们可以从elems中找到我们定义的Object

- People -> `*TypeName`
- main -> `*Func`

## imports

`imports`帮我们关联了导入的package

还是以上面程序为例，它引用了`fmt`

```
(dlv) p pkg.imports
[]*go/types.Package len: 1, cap: 1, [
	*{
		path: "fmt",
		name: "fmt",
		scope: *(*"go/types.Scope")(0xc00022c000),
		complete: true,
		imports: []*go/types.Package len: 5, cap: 6, [
			*(*"go/types.Package")(0xc00022c0f0),
			*(*"go/types.Package")(0xc00022c190),
			*(*"go/types.Package")(0xc00022c230),
			*(*"go/types.Package")(0xc00022c2d0),
			*(*"go/types.Package")(0xc00022c370),
		],
		fake: false,},
]
```

我们可以看出，`fmt`也引用了其它的package

## 小结一下

Package提供了

- package的scope
- 其它被这个package引用的package

是一个比较常用的入口