goast
=====

goast is a Go AST (Abstract Syntax Tree) utility with the aim of providing idiomatic generic programming facilities by taking advantage of Go's native AST abilities and the go:generate build directive to enable intelligent code generation.

goast's core philosophies are

* Compile time type safety
* No runtime type casting (see previous)
* Avoid runtime reflection
* Prefer pure Go over syntax extensions
* No text templates (see previous)
* Prefer inference over annotation
* Dependency free

The functionality of goast is currently built on the following axiom and proposition

1. The empty interface (`interface{}`) can be replaced with any other type
2. Any composite type composed at least partially of the empty interface (e.g. `map[string]interface{}`) can be replaced with any other composite type of the same structure with the empty interface swapped out for a concrete type (e.g. `map[string][]int64`)

## Simple example

Consider the following generic implementation of a Filter method on a slice


```go
//file: slicefilter.go
package gen

type T interface{}
type Slice []T
func (s Slice) Filter(fn func(T)bool) (result Slice) {
	for _, v := range s {
		if fn(v) {
			result = append(result, v)
		}
	}
	return
}
```

This code compiles, and accurately reflects the algorithm that any slice type might implement. It is, however, unusable for any type without some amount of type casting, or by a developer providing a concrete implementation per slice type.

With goast, a developer only needs to provide the following code, and the go generate command will provide the rest

```go
//file main.go
package main

//go:generate goast write impl slicefilter.go

type Ints []int
```

The `go:generate` build directive instructs goast to write an implementation of the code in slicefilter.go for the types provided in main.go, resulting in the following file being generated

```go
//file ints_slicefilter.go
package main

func (s Ints) Filter(fn func(int)bool) (result Ints) {
	for _, v := range s {
		if fn(v) {
			result = append(result, v)
		}
	}
	return
}
```

A more complete set of slice operations can be seen in the [sliceops](https://github.com/jamesgarfield/sliceops) package.


## Complex Example

Vanilla iteration patterns aren't exciting enough for you? 

Want something more Go-centric, maybe even concurrency based? 

How about a Fan-Out/Fan-In Concurrent Pipeline?

If you're not familiar with pipelines, [this](http://blog.golang.org/pipelines) is a great primer on the pattern and pitfalls of implementing it. The concept is strightforward, but it's not the kind of thing I can see trusting myself to write repeatedly.

To see how it's possible to abstract this complex pattern with goast, refer to the [Pipeline](https://github.com/jamesgarfield/goast/tree/master/examples/pipeline) example located in this repository.


## Roadmap

goast is still in an alpha/RFC stage of development. Some features that are planned for v1 are

* ~~Support for generic structs~~ (done) ~~[Issue](https://github.com/jamesgarfield/goast/issues/1)~~
* ~~Related Types~~ (done) ~~[Issue](https://github.com/jamesgarfield/goast/issues/3)~~
* Projection [Issue](https://github.com/jamesgarfield/goast/issues/4)
* Inferred renaming. [Issue](https://github.com/jamesgarfield/goast/issues/2)
* Pruning. [Issue](https://github.com/jamesgarfield/goast/issues/6)
* Support for comments. [Issue](https://github.com/jamesgarfield/goast/issues/5)
* File naming control [Issue](https://github.com/jamesgarfield/goast/issues/7)


## History and acknowledgements

I originally got interested in code generation as a method of genericty in Go when I learned about the [gen](http://clipperhouse.github.io/gen/) package from clipperhouse. When to Go team first announced Go 1.4 and the go:generate proposal, it planted the seed of the idea for goast in my brain and initiated my research into how it might work. In the intervening time I found [gotgo](https://github.com/droundy/gotgo), and more recently (and also quite close to my goals) the [gonerics](https://github.com/bouk/gonerics) package. As projects in the same area as what goast explores, they were all valuble for research and inspiration, as well for providing a contrast against which I wanted to differentiate.

I'd also like to specificaly thank Rob Pike for for being such a staunch believer in a world of generic programming without language level generics. Without all the constant assertions that there were "other ways", I most likely would not have started down this path.

