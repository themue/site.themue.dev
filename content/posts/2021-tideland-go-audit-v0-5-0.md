---
title: "Tideland Go Audit v0.5.0"
date: "2021-05-20T22:25:00+01:00"
draft: false
categories: ["development"]
tags: ["golang", "tideland", "library", "testing"]
---

I've mentioned before that I'm currently reorganize my **Tideland Go Libraries** - once again. I'm sorry for it and it will be interesting just for those who want to participate in development. The import into your own projects will stay the same.

This time it's about the **Tideland Go Audit** library, I've just released the new **v0.5.0**. It's a well approved and robust library containing helpful packages for testing. So the update only contains a few changes.

* The main package `asserts` has been extended by the additional assertions `NotOK()` and `AnyError()`.
* In case of asserting tests and using `Failable` which like `testing.T` does define `Logf()` and `Errorf()` their output is more compatible with CI environments.
* The `generators` package has been extended by a `OneOf()` function returning anyone of the given variadic parameters randomly.

In total the **Tideland Go Audit** library helps you to test your projects in multiple ways.

* `asserts` contains many descriptive assertions not only usable for unit tests.
* `capture` helps capturing `stdout` and `strerr` for tests.
* `environments` simplifies setting of environment variable and creation of temporary directories. It also can run a small web server for tests.
* `generators` contains generating test data in many ways. It can be randomly or fixed for repeatability.
