# conjungo

[![LICENSE](https://img.shields.io/badge/license-MIT-orange.svg)](LICENSE)
[![Golang](https://img.shields.io/badge/Golang-v1.7-blue.svg)](https://golang.org/dl/)
[![Godocs](https://img.shields.io/badge/golang-documentation-blue.svg)](https://godoc.org/github.com/InVisionApp/conjungo)
[![Go Report Card](https://goreportcard.com/badge/github.com/InVisionApp/conjungo)](https://goreportcard.com/report/github.com/InVisionApp/conjungo)
[![Travis Build Status](https://travis-ci.com/InVisionApp/conjungo.svg?token=KosA43m1X3ikri8JEukQ&branch=master)](https://travis-ci.com/InVisionApp/conjungo) 
[![codecov](https://codecov.io/gh/InVisionApp/conjungo/branch/master/graph/badge.svg?token=lesB1PUEtL)](https://codecov.io/gh/InVisionApp/conjungo)

A merge utility designed for flexibility and customizability. The library has a
simple interface that uses a set of default merge functions that will fit most
basic use cases. From there, specific customizations can be made to merge things
in any particular way that is needed.

Merge any two things of the same type, including maps, slices, and structs. By
default, the target value will be overwritten by the source. If the overwrite
option is turned off, only new values in source that do not already exist in
target will be added.  
If you would like to change the way two items of a particular type get merged,
custom merge functions can be defined for any type or kind (see below).  

## Why Conjungo?

The definition of Conjungo: 
```
I.v. a., to bind together, connect, join, unite (very freq. in all perr. and species of composition); constr. with cum, inter se, the dat., or the acc. only; trop. also with ad.
```
Reference: [Latin Dictionary...](http://www.perseus.tufts.edu/hopper/text?doc=Perseus:text:1999.04.0059:entry=conjungo)

## Setup
In order to use **conjungo**, you should vendor it within your project.

```sh 
govendor fetch github.com/InVisionApp/conjungo
```

## Usage
Merge two structs together:
```go
type Foo struct {
	Name    string
	Size    int
	Special bool
}

targetStruct := Foo{
	Name:    "target",
	Size:    2,
	Special: false,
}

sourceStruct := Foo{
	Name:    "source",
	Size:    4,
	Special: true,
}

merged, err := conjungo.Merge(targetStruct, sourceStruct, nil)
if err != nil {
	log.Error(err)
}
```

Merge two `map[string]interface{}` together:
```go
targetMap := map[string]interface{}{
	"A": "wrong",
	"B": 1,
	"C": map[string]string{"foo": "unchanged", "bar": "orig"},
}

sourceMap := map[string]interface{}{
	"A": "correct",
	"B": 2,
	"C": map[string]string{"bar": "newVal", "safe": "added"},
}

// use the map merge wrapper
merged, err := conjungo.MergeMapStrIface(targetMap, sourceMap, nil)
if err != nil {
	log.Error(err)
}

// OR 

// use the main merge func
merged, err := conjungo.Merge(targetMap, sourceMap, nil)
if err != nil {
	log.Error(err)
}
mergedMap, _ := merged.(map[string]interface{})
```

Define a custom merge function for a type:
```go
opts := conjungo.NewOptions()
opts.MergeFuncs.SetTypeMergeFunc(
	reflect.TypeOf(0),
	// merge two 'int' types by adding them together
	func(t, s interface{}, o *conjungo.Options) (interface{}, error) {
		iT, _ := t.(int)
		iS, _ := s.(int)
		return iT + iS, nil
	},
)

merged, err := conjungo.Merge(1, 2, opts)
if err != nil {
	log.Error(err)
}
// merged == 3
```

or for a kind:
```go
opts := conjungo.NewOptions()
opts.MergeFuncs.SetKindMergeFunc(
	reflect.TypeOf(struct{}{}).Kind(),
	// merge two 'struct' kinds by replacing the target with the source
	func(t, s interface{}, o *conjungo.Options) (interface{}, error) {
		return s, nil
	},
)
```

See [examples](example/main.go) for more details.
