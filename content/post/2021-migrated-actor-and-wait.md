+++
date = "2021-09-08T19:00:00+01:00"
draft = false
title = "Migrated Go Actor and Go Wait"
tags = ["golang", "tideland", "package", "actor", "wait"]
categories = ["tideland"]
+++

As described a few days ago, I am migrating individual packages of the Tideland
libraries to owb repositories. I have already described my motivation for this.
In the meantime, two more packages have been migrated to the main level:
**Tideland Go Actor** and **Tideland Go Wait**.

The package **Tideland Go Actor** pursues the idea to realize concurrency in Go
not only via CSP, but also with the *Actor Model*. It picks up a bit the concept
of the Erlang/OTP module `gen_server`. There data is sent synchronously or asynchronously
to the process, here processed serialized and then eventually responses are sent back
to the sender. In the Go package `actor`, however, closures are processed serialized
by the goroutine. From the developer's point of view, the business logic and the use
of concurrency thus remain closer together and more transparent.

Often there are situations where a condition is checked, but this is not done with a
simple `if`. It is then much more necessary to repeat the checks until the desired
condition or a knockout condition has occurred. The latter can be a termination from
outside, the reaching of a maximum number of checks, a timeout or a deadline. Also
the test can say that it makes no sense to continue testing. It can also be important
to define the frequency of the tests, to let them flutter a bit to avoid collisions,
or a completely individual and dynamic timing of the tests. All this is offered by
the **Tideland Go Wait** package.

