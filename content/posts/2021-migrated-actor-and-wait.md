---
title: "Migrated Go Actor and Go Wait"
date: "2021-09-08T21:11:00+01:00"
draft: false
categories: ["development"]
tags: ["golang", "tideland", "library", "actor", "wait"]
---

As described a few days ago, I am migrating individual packages of the Tideland libraries to own repositories. I’ve described my motivation for this before. In the meantime, two more packages have been migrated to the main level: **Tideland Go Actor** and **Tideland Go Wait**.

The package [Tideland Go Actor](https://pkg.go.dev/tideland.dev/go/actor) pursues the idea to realize concurrency in Go not only via CSP, but with the **Actor Model**. It picks up the concept of the Erlang/OTP module `gen_server`. There data is sent synchronously or asynchronously to the process, here processed serialized and then eventually responses are sent back to the sender. In the Go package `actor`, however, closures are processed serialized by the goroutine. From the developer's point of view, the business logic and the use of concurrency thus remain closer together and more transparent.

Sometimes there are situations where a condition has to be checked, but not only with a simple `if` statement. It is  much more necessary to repeat these checks until the desired condition or a knockout condition has occurred. The latter can be a termination from outside, the reaching of a maximum number of checks, a timeout or a deadline. Also the test can say that it makes no sense to continue testing. It may be important to define the frequency of the tests, to let them flutter to avoid collisions, or an own individual and dynamic timing of the tests. All this is offered by the [Tideland Go Wait](https://pkg.go.dev/tideland.dev/go/wait) package.

