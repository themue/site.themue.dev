+++
date = "2019-10-13T12:00:00+01:00"
draft = false
title = "Tideland Go Library v0.2.0"
tags = ["golang", "tideland", "library", "release"]
categories = ["tideland"]
+++

Here it is, a new minor release of the **Tideland Go Library**. It's the new **v0.2.0** and contains some smaller changes. You can read about it inside the [changelog](https://github.com/tideland/go/blob/v0.2.0/CHANGELOG.md).

But it also contains two bigger blocks. One has been the splitting of the somehow poor designed and organized `webbex` into the new `httpx` and `web`. The other one is the even bigger one, I needed most of the time for it. It's the re-engineering of the `cells` package for event-driven Go applications.

The first ideas for the next release are already planned and noted as [issues](https://github.com/tideland/go/issues). Quite interesting will be the factories for typical cell meshes. Also thinking about adding more helper packages for the work with concurrency.
