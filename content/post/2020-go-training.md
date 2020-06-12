+++
date = "2020-06-11T10:15:00+01:00"
draft = false
title = "Working Title 'Go to Go'"
tags = ["golang", "taining", "online"]
categories = ["development"]
+++

Last Monday I had a remote workshop about Go programming. It has been an interesting experience, not yet done something similar before. Sadly I needed too much time for the basic topics. So I had not enough time in the end to dive into my example. This has been a module containing a little declarative environment for components, a bit like Kubernetes. So to use this I decided to create an online training for Go based on an improved version of that example.

The idea is to provide everything as a GitHub project. Planned are a `doc` directory containing an introduction into the history of Go, the people behind it, and the tools. Additionally it will contain a quick introduction into Go to learn the most important structures, data types, and statements. But only so much to understand the following samples.

The main one will be `gube`. That's the little environment for concurrently running Go components. While they - due to Gos nature - late have to be statically linked their usage can be dynamically configured, together with storage, events, queues, and configuration. Kubernetes users will recognize those kind of components. The interesting part there will be, that the documents of `doc` lead into the intividual directories of `gube` step by step. Documents there will introduce into the tasks and ideas of the individual package and then link into the source code. All files their will contain lots of didictical comments helping to understand how Go works and why solutions are done like in the example.

Second module will be `samples`. It will use `gube`, contain a number of components, create a deployments, and so show how everything works together. Like in `gube` the documents in `doc` lead the solutions, package documents introduce them, and lots of comments shall help to understand Go.

It's an experiment. You can just read it online, you can clone and try it, you can do changes to experiment, and if you want to participate you can create pull request.