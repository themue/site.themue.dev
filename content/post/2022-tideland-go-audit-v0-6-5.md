---
title: 'Tideland Go Audit v0.6.5'
date: '2022-03-01T15:43:00+01:00'
draft: false
categories: ['tideland']
tags: ['golang', 'tideland', 'library', 'release', 'testing']
---

There are times when you find a bug in your software. Today it had been in my testing
library **TIdeland Go Audit**. Here the assertion `ErrorContains()` reacted with a panic
in case of a `nil` error. So I fixed it like I already had done it earlier in `ErrorMatch()`.
Interestingly I found in testing that I didn't verified it there. So this test is now also changed.

Additionally during tests for a different library with high concurrency I, or better `go test` during
the tests, discovered a race condition. I'm using `math.Rand` in `generators.Generator` and it is not
safe for concurrent use. In most cases, just as it is a simple test data generater, it has been okay.
But now I wanted to use this opportunity and so I fixed this bug too.

The documentation still can be found at [pkg.go.dev](https://pkg.go.dev/tideland.dev/go/audit), the
project sources at [GitHub](https://github.com/tideland/go-audit). Enjoy it.

