+++
date = "2020-06-12T10:15:00+01:00"
draft = false
title = "Working Title 'Go to Go'"
tags = ["golang", "taining", "online"]
categories = ["development"]
+++

Last Monday I had a remote workshop about Go programming. It has been an interesting experience, not yet done something similar before. Sadly I needed too much time for the basic topics. So I had not enough time in the end to dive into my example. This has been a module containing a little declarative environment for components, a bit like Kubernetes. As a result I decided to create an online training for Go based on an improved version of that example.

The idea is to provide everything as a GitHub project. Planned inside are a `doc` directory containing an introduction into the history of Go, the people behind it, and the tools. Additionally it will contain a quick introduction into the language Go to learn the most important structures, data types, and statements. But only so much to get a first understanding of the following samples.

The main one will be the module `gube`. That's the little environment for concurrently running Go components. Those are based on the interface `Runnable` the user has to implement for own logic. They can be deployed together with ressources like `Config`, `Storage`, or `Queue` using the API of the central `Manager`. The interesting part there will be, that the documents of `doc` lead into the intividual directories of `gube` step by step. Documents there will introduce into the tasks and ideas of the individual packages and then link into the source code. All files their will contain lots of didactic comments helping to understand how Go works and why solutions are done like in the example.

Second module will be `samples`. It will use `gube`, contain a number of components, create the matching deployments, and possibly some binaries as runtime for all. That way I'll try to show how everything works together. Like in `gube` the documents in `doc` lead the solutions, package documents introduce them, and lots of comments will help to understand Go. Here I'm open for any good ideas for applications based on `gube`.

It's an experiment. You can just read it online, you can clone and try it, you can do changes to experiment, and if you want to participate you can create pull request. I hope you'll enjoy it.
