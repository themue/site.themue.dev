---
title: 'Tideland Go Actor v0.3.0'
date: '2023-04-09T15:00:00+01:00'
draft: false
categories: ['tideland']
tags: ['golang', 'tideland', 'library', 'release', 'actor']
---

Hey there! I'm happy to share with you some news about the new release v0.3.0 of the **Tideland Go Actor**. This new version comes with a handful of exciting updates and improvements that make it easier and more efficient to work with Actors in Go. One important change in this release is the addition of the `Repeat()` method. This new feature allows developers to run background Actions in intervals, making it easier to handle long-running tasks in the background.

Additionally, the **Tideland Go Actor** now added `context.Context` to individual Action calls, which provides developers with better control. This is a significant improvement as it allows for more granular control like timeouts, deadlines or cancellations. As the context now adds more control the both methods `DoSyncTimout()` and `DoAsyncTimeout()` have been removed.

Another internal change is, that synchronous and asynchrounous Actions are now handled in the same queue. In this form, the synchronous Actions are no longer preferred or can even block asynchronous Actions. Lastly, the **Tideland Go Actor** has improved external checking to see if the Actor is still running by adding a `IsDone() bool` method as well as a `Done() <-chan struct{}` method.

Overall, the **Tideland Go Actor v0.3.0** release includes a host of valuable updates and improvements that will make working with Actors in Go easier and more efficient. For more information, please see the [documentation](https://pkg.go.dev/tideland.dev/go/actor) and [source code](https://github.com/tideland/go-actor). I hope it helps you to creat concurrent applications in Go a more simple way.
