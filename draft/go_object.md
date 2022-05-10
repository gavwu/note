# Go元编程系列--Object

`Object`似乎人人都懂，毕竟是现在还是面向对象编程的时代

但是Go的`Object`可能并不像其它语言（有些语言把Object当成所有对象的基类）

## pre-requisites

- [Go元编程系列--Type](go_type.md)
- [Go元编程系列--Scope](go_scope.md)
- [Go元编程系列--Package](go_package.md)


## Object介绍

```go
// An Object describes a named language entity such as a package,
// constant, type, variable, function (incl. methods), or label.
// All objects implement the Object interface.
//
type Object interface {
	Parent() *Scope // scope in which this object is declared; nil for methods and struct fields
	Pos() token.Pos // position of object identifier in declaration
	Pkg() *Package  // package to which this object belongs; nil for labels and objects in the Universe scope
	Name() string   // package local object name
	Type() Type     // object type
	Exported() bool // reports whether the name starts with a capital letter
	Id() string     // object name if exported, qualified name if not exported (see func Id)

	// String returns a human-readable string of the object.
    String() string
    
    ...
}
```

从注释上看，Go的Object有：

- Package（指的是引用的Package，比如一个go文件引用了`fmt`）
- 常量
- 自定义类型（具体为`TypeName`，类似`type A int`）
- 变量
- 函数
- label（这是在使用`goto`时候需要指定的Label）

从实现上，还多出了：

- *Nil*
- *Builtin*

如果Object是`Field`(一般是`*Var`)，或者是`Method`(一般是`*Func`)，它们的Parent为`nil`

> 需要注意的是，这个接口提供的`Id()`其实并不唯一



## Common Object

先来看一下每一个Object都会继承的基类

```go
// An object implements the common parts of an Object.
type object struct {
	parent    *Scope
	pos       token.Pos
	pkg       *Package
	name      string
	typ       Type
	...
}
```

> 此处省略了一些内部字段（仅供内部使用）

从字面上来解释这些字段所带来的作用

- parent，这个object被定义的作用域
- pos，object被**声明**的起始位置
- pkg，object所属的package（对于`a := 1`里a这个object，pkg为`nil`）
- name，名字（unqualified）
- typ，类型


## PkgName

```go
// A PkgName represents an imported Go package.
// PkgNames don't have a type.
type PkgName struct {
	object
	imported *Package
	used     bool // set if the package was used
}
```

PkgName还提供了被导入的Package信息

## Const

```go
// A Const represents a declared constant.
type Const struct {
	object
	val constant.Value
}
```

多了一个常量的Value

## TypeName

```go
// A TypeName represents a name for a (defined or alias) type.
type TypeName struct {
	object
}
```

这个就是我们平时用

```go
type A int
```

所得到的

## Var

```go
// A Variable represents a declared variable (including function parameters and results, and struct fields).
type Var struct {
	object
	embedded bool // if set, the variable is an embedded struct field, and name is the type name
	isField  bool // var is struct field
	used     bool // set if the variable was used
}
```

这个从注释上也非常易懂，包含了变量中的一些属性

- 是否内嵌
- 是否为字段
- 是否使用

从下面三个函数，可以看出Go对Var不同使用方式的设计

```go
// NewVar returns a new variable.
// The arguments set the attributes found with all Objects.
func NewVar(pos token.Pos, pkg *Package, name string, typ Type) *Var {
	return &Var{object: object{nil, pos, pkg, name, typ, 0, colorFor(typ), token.NoPos}}
}

// NewParam returns a new variable representing a function parameter.
func NewParam(pos token.Pos, pkg *Package, name string, typ Type) *Var {
	return &Var{object: object{nil, pos, pkg, name, typ, 0, colorFor(typ), token.NoPos}, used: true} // parameters are always 'used'
}

// NewField returns a new variable representing a struct field.
// For embedded fields, the name is the unqualified type name
/// under which the field is accessible.
func NewField(pos token.Pos, pkg *Package, name string, typ Type, embedded bool) *Var {
	return &Var{object: object{nil, pos, pkg, name, typ, 0, colorFor(typ), token.NoPos}, embedded: embedded, isField: true}
}
```

比如说

- Param的`used`永远都是`true`
- 只有字段才会设置`embedded`


## Func

```go
// A Func represents a declared function, concrete method, or abstract
// (interface) method. Its Type() is always a *Signature.
// An abstract method may belong to many interfaces due to embedding.
type Func struct {
	object
	hasPtrRecv bool // only valid for methods that don't have a type yet
}
```

从注释的信息我们得知，`Func`的`Type`只会是`Signature`

一个非常有用的函数

```go
// FullName returns the package- or receiver-type-qualified name of
// function or method obj.
func (obj *Func) FullName() string {
	var buf bytes.Buffer
	writeFuncName(&buf, obj, nil)
	return buf.String()
}
```

我们再对比一些函数的时候，可以直接用

```go
if obj.FullName() == "fmt.Println" {
    //...
}
```

## Label

```go
// A Label represents a declared label.
// Labels don't have a type.
type Label struct {
	object
	used bool // set if the label was used
}
```

多出了一个`used`，说明Go是靠这个来检测一个Label是否被用过。

