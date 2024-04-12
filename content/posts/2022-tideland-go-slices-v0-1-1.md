---
title: "Tideland Go Slices v0.1.1"
date: "2022-08-20T21:00:00+01:00"
draft: false
categories: ["development"]
tags: ["golang", "tideland", "library", "release", "slices", "generics", "erlang", "lists"]
---

So far I had not missed **generics** in **Go**. Much could be solved via interfaces or closures. And with the disgusting syntactic aberrations of other languages when dealing with generics, I was rather afraid that the elegant simplicity of Go could be lost. As of Go 1.18, Google's language now handles generics after much discussion.

I have always appreciated the implicit simplicity with which, for example, types in a statement like

```go
imAString := myStringReturningFunc(some, "input")
```

can be derived. Why should I specify the type of `imAString` too? It is the return type of `myStringReturningFunc()` , so this clear. It feels like in dynamically typed languages. And it is nice that we managed to transfer this ease to the generics as well. Now it's just a matter of not polluting your own code with unnecessary generics and possibly used constraints. 

Nevertheless, Go version 1.19 has been released in the meantime and I had not yet found a use for generics. But I wanted to give it a try. Then I thought of the feature-rich module `lists` in the **Erlang/OTP** language. I wanted to use this for slices in Go. The result is my new library **Tideland Go Slices v0.1.1**. Thanks to generics it allows easy and comfortable handling of slices of any type. Some instructions would be for example

```go
vs1 := slices.FoldL(func(s string, i int) int { return len(s) + i }, 0, myStringSlice)
vs2 := slices.Reverse(myCustomerStructSlice)
vs3 := slices.SortWith(func(cs []*Customer, i, j int) {
    return cs[i].Name < cs[j].Name
}, myCustomerStructSlice)
vs4 := slices.Unique(mySliceWithDuplicates)
```

Documentation and code can be found at

* [https://pkg.go.dev/tideland.dev/go/slices](https://pkg.go.dev/tideland.dev/go/slices) and
* [https://github.com/tideland/go-slices](https://github.com/tideland/go-slices).

Have fun with it.