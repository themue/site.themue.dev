+++
date = "2021-05-30T20:00:00+01:00"
draft = false
title = "Tideland Go Audit v0.5.0"
tags = ["golang", "tideland", "library", "release", "testing"]
categories = ["tideland"]
+++

While step by step  reorganizing the Tideland Go libraries the one for testing
your projects reached a new version. I've just released the *Tideland Go Audit v0.5.0*.
As it already is a very robust and complete library there are only few changes.

* The `asserts` package now provides the additional assertions `NotOK()` and `AnyError()`.
* In case of asserting tests and using `Failable` like `testing.T` their provided `Logf()`
  and `Errorf()` will be used for output. This way there are less conflicts with some
  CI environments.
* The `generators` package now also contains a `OneOf()` returning one of the given
  variadic parameters.

The library *Tideland Go Audit* supports testing Go projects in multiple ways:

* Package `asserts` provides functions for assertions helpful in tests and validation.
* Package `capture` allows capturing of STDOUT and STDERR.
* Package `environments` provides setting of environment variables, creation of temporary
  directories, and running web servers for tests.
* Package `generators` simplifies generation of test data. In case of initializing it
  with a fixed random the generated values are also repeatable.

Documentation can be found at [pkg.go.dev](https://pkg.go.dev/tideland.dev/go/audit), the
project sources at [GitHub](https://github.com/tideland/go-audit). Enjoy it.

