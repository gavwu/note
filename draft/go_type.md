
# Go Type介绍

go/types/type.go
```go
// A Type represents a type of Go.
// All types implement the Type interface.
type Type interface {
	// Underlying returns the underlying type of a type.
	Underlying() Type

	// String returns a string representation of a type.
	String() string
}
```

Go的所有类型都实现了这个接口，在做一些Go的元编程的时候，你需要了解这个接口的各个具体实现。

通过了解这些类型，还能知道Go是如何看待这些类型的，以及如何设计它们的。通过它的设计，我们也能知道一些“真理”。

## Basic

`Basic`是一个比较有意思的类型，从它的字面可以看出它是我们常用Go的基本类型

```go
// A Basic represents a basic type.
type Basic struct {
	kind BasicKind
	info BasicInfo
	name string
}
```

其中我们可以通过`kind`来得知它的具体类型，比如`string`，通过`name`来得知这个变量名字。

```go
const (
	Invalid BasicKind = iota // type is invalid

	// predeclared types
	Bool
	Int
	Int8
	Int16
	Int32
	Int64
	Uint
	Uint8
	Uint16
	Uint32
	Uint64
	Uintptr
	Float32
	Float64
	Complex64
	Complex128
	String
	UnsafePointer

	// types for untyped values
	UntypedBool
	UntypedInt
	UntypedRune
	UntypedFloat
	UntypedComplex
	UntypedString
	UntypedNil

	// aliases
	Byte = Uint8
	Rune = Int32
)
```

### Invalid

> the kind Invalid indicates the invalid type, which is used for expressions containing errors, or for objects without types, like `Label`, `Builtin`, or `PkgName`.

有一些是没有类型的，像`Label`，所以会有一个`Invalid`


### Normal Type


从`Bool`到`UnsafePointer`，有很多我们常用的类型，只不过像`Uintptr`和`UnsafePointer`对于我们在上层做应用的，可能很少用

### Interesting Untyped

以`Untyped`开头的这几个（除了`UntypedNil`）是我们在定义常量时候的类型

```go
const myConst = 1 //UntypedInt
```

为什么我们需要这个？直接用string类型来表示不就行了吗？

有这样疑问的可以参考下面代码，也可以[试试](https://play.golang.org/p/69Ri_sFxG-7)

```go
package main

import (
	"fmt"
)

type MyString string

// untype string
const untypeString = "Hello, World!"

// type is string
const typeString string = "Hello, World!"

func main() {
	var m MyString
	// success assignment
	m = untypeString
	// m = typeString // Compile Error
	fmt.Println(m)
}
```

从上面这个例子我们也可以知道Go是一个强类型的语言，即使MyString和string的底层一样，但也不允许直接赋值。

Go当然也意识到了这样做的不方便之处，因此提出了Constant，完全是为了开发者能够减少编写类型转换的代码。

> 关于Go的Constant设计，可以参考官方[blog](https://blog.golang.org/constants)，从这个可以Go在设计Constant时的考量


还有一些`别名`，所以我们很简单就验证了

- `Byte`就是`Uint8`
- `Rune`就是`Int32`

## Array

```go
// An Array represents an array type.
type Array struct {
	len  int64
	elem Type
}
```

> 注意Array不是Slice

很简单，只有一个长度`len`，和类型`elem`，很好理解。我们还可以进一步分析`elem`的类型

> 所以我们又能验证一个真理：Go的数组的类型比较是包括了**长度**

## Slice

```go
type Slice struct {
	elem Type
}
```

和`Array`相比，少了个`len`，说明`Slice`的类型是没有**长度**

## Struct

```go
type Var struct {
	object
	embedded bool // if set, the variable is an embedded struct field, and name is the type name
	isField  bool // var is struct field
	used     bool // set if the variable was used
}

// A Struct represents a struct type.
type Struct struct {
	fields []*Var
	tags   []string // field tags; nil if there are no tags
}
```

可以看出，Go是将结构体分成了`变量`和`Tag`

```go
type A struct {
	a int `json:"a"`
}
```

还可以看出，我们还能通过具体的变量看到这个字段：

- 是否为内嵌
- 是否被使用（有没有很熟悉的感觉）

## Pointer

```go
// A Pointer represents a pointer type.
type Pointer struct {
	base Type // element type
}
```

一个指针指向一块内存地址，这块内存地址存储的数据属于某一个类型（`base`）

## Tuple

```go
// A Tuple represents an ordered list of variables; a nil *Tuple is a valid (empty) tuple.
// Tuples are used as components of signatures and to represent the type of multiple
// assignments; they are not first class types of Go.
type Tuple struct {
	vars []*Var
}
```

从官方的comment可以知道，`Tuple`可以作为

- 函数的`Params`和`Results`
- 一次为多个变量赋值


## Signature

```go
// A Signature represents a (non-builtin) function or method type.
// The receiver is ignored when comparing signatures for identity.
type Signature struct {
	// We need to keep the scope in Signature (rather than passing it around
	// and store it in the Func Object) because when type-checking a function
	// literal we call the general type checker which returns a general Type.
	// We then unpack the *Signature and use the scope for the literal body.
	scope    *Scope // function scope, present for package-local signatures
	recv     *Var   // nil if not a method
	params   *Tuple // (incoming) parameters from left to right; or nil
	results  *Tuple // (outgoing) results from left to right; or nil
	variadic bool   // true if the last parameter's type is of the form ...T (or string, for append built-in only)
}
```

针对其注释解释一下：

- Signature是一种non-builtin函数的类型
- scope代表着这个函数的作用域
- recv代表着接受者（如果有）
- params，results就是入参出参
- variadic表明函数最后一个参数是否为`...`形式

**这里可能会有疑问**：为什么一个函数没有名字？

- 因为这里只是描述一个签名，并不是一个函数，函数在Go里是一个`Object`，并不是一个`Type`

> Object会在另一篇文章写到

## Interface

```go
// An Interface represents an interface type.
type Interface struct {
	methods   []*Func // ordered list of explicitly declared methods
	embeddeds []Type  // ordered list of explicitly embedded types

	allMethods []*Func // ordered list of methods declared with or embedded in this interface (TODO(gri): replace with mset)
}
```

这里也非常好理解，一个`Interface`可以定义多个方法，也可以嵌套多个`Interface`

> 其中这里有关Interface Complete的概念可以自行探讨

### 有用的方法

标准库中好提供了一些好用的API帮助我们分析Interface

```go
func IsInterface(Type) bool
func Implements(V Type, T *Interface) bool
func AssertableTo(V *Interface, T Type) bool
func MissingMethod(V Type, T *Interface, static bool) (method *Func, wrongType bool)
```

## Map

```go
// A Map represents a map type.
type Map struct {
	key, elem Type
}
```

描述一个Map只需要描述Key和Elem的类型

## Chan

```go
// A Chan represents a channel type.
type Chan struct {
	dir  ChanDir
	elem Type
}

// A ChanDir value indicates a channel direction.
type ChanDir int

// The direction of a channel is indicated by one of these constants.
const (
	SendRecv ChanDir = iota
	SendOnly
	RecvOnly
)
```

Channel只需要定义其`方向`和`类型`即可

## Named**

```go
// A Named represents a named type.
type Named struct {
	obj        *TypeName // corresponding declared object
	underlying Type      // possibly a *Named during setup; never a *Named once set up completely
	methods    []*Func   // methods declared for this type (not the method set of this type); signatures are type-checked lazily
}
```

Named类型是一个比较有意识的类型，何为Named？

```go
package main

import "fmt"

type People struct {
	age int
	name string
}

type MyMap = map[string]string

func main() {
        fmt.Println("Hello, world")
}
```

首先先来看整体情况

```
(dlv) p pkg.scope.elems
map[string]go/types.Object [
	"People": *go/types.TypeName {
		object: (*"go/types.object")(0xc0002d2b40),},
	"MyMap": *go/types.TypeName {
		object: (*"go/types.object")(0xc0002d2b90),},
	"main": *go/types.Func {
		object: (*"go/types.object")(0xc0002d2be0),
		hasPtrRecv: false,},
]
```

在package的scope下，我们可以看到3个elem，分别是

- People，定义的结构体
- MyMap，定义的别名
- main，函数


我们先看People

```
(dlv) p pkg.scope.elems["People"].typ
go/types.Type(*go/types.Named) *{
	obj: *go/types.TypeName {
		object: (*"go/types.object")(0xc0002d2b40),},
	underlying: go/types.Type(*go/types.Struct) *{...},
	methods: []*go/types.Func len: 0, cap: 0, nil,}
```

可以看出People的`Type`就是`Named`

再来看`MyMap`这个

```
(dlv) p pkg.scope.elems["MyMap"].typ
go/types.Type(*go/types.Map) *{
	key: go/types.Type(*go/types.Basic) *{kind: String (17), info: IsString (32), name: "string"},
	elem: go/types.Type(*go/types.Basic) *{kind: String (17), info: IsString (32), name: "string"},}
```

不是`Named`而是`Map`

所以我们可以区分出，带`=`的，是别名，定义出来的`Object.Type()`是和`=`后面的保持一致

不带`=`的是一种定义，`Object.Type()`是`Named`


### 为什么叫Named？

其实这个名字起的确实让人疑惑，在官方的例子中有对这个作出解释

> Since Go 1.9, the Go language specification has used the term defined types instead of named types; the essential property of a defined type is not that it has a name, but that it is a distinct type with its own method set. However, the type checker API predates that change and instead calls defined types "named" types.

大概意思就是，因为Go1.9引入了`别名`，但是因为`type checker`在这之前就已经实现了，所以就还是叫这个名字



## 所有类型的Underlying

```go
func (b *Basic) Underlying() Type     { return b }
func (a *Array) Underlying() Type     { return a }
func (s *Slice) Underlying() Type     { return s }
func (s *Struct) Underlying() Type    { return s }
func (p *Pointer) Underlying() Type   { return p }
func (t *Tuple) Underlying() Type     { return t }
func (s *Signature) Underlying() Type { return s }
func (t *Interface) Underlying() Type { return t }
func (m *Map) Underlying() Type       { return m }
func (c *Chan) Underlying() Type      { return c }
func (t *Named) Underlying() Type     { return t.underlying }
```

可以看出，除了`Named`，其它类型都已经是其底层的类型

到了底层的类型，我们就可以通过底层类型所拥有的函数进行进一步分析，如：

```go
// Elem returns the element type of slice s.
func (s *Slice) Elem() Type { return s.elem }
```

如果我们断言一个类型为`Slice`，我们可以具体看到元素的类型。

```go
if slice, ok := t.(*Slice); ok {
	return slice.Elem()
}
```

## 递归的设计

细心的人可能已经发现了，这里的类型很多时候都是递归嵌套的，比如

- Slice的elem是Pointer
- Pointer的base是Struct
- Struct...

这其实说明，我们在做元数据编程的时候，不能按照流程式来分析，因为有可能会因为复杂的递归规则，搞得头晕转向，最后写出的代码只能是无法通用


## 小结一下

Go Type的定义其实和我们平时写代码时所见到的基本一样，熟悉写Go代码的同学肯定能够`反射`回自己写的代码

对Type的了解帮助我们更好地从Go语言本身去看待这些类型，而不是仅仅从语法上看

