+++
date = "2021-08-29T13:00:00+01:00"
draft = false
title = "Make it easier to find"
tags = ["golang", "tideland", "package"]
categories = ["tideland"]
+++

A look at the [Tideland Go repositories](https://github.com/tideland/) shows that these today
are libraries for individual topics with various packages included. One problem with this form
of organization is that these packages are difficult to find. At the same time, their individual
histories and their versions are tied to those of the entire library. This must be improved.

Therefore, the reconstruction of these projects has now begun. So some of the packages from the
repositories move to the main level and become independent projects. A first example is now
[Tideland Go UUID](https://github.com/tideland/go-uuid). The package implements the generation
of the versions v1, v3, v4, and v5. v2 will follow, as well as v6, v7, and v8 when the corresponding
[draft](https://datatracker.ietf.org/doc/html/draft-peabody-dispatch-new-uuid-format) is approved.

In which order it goes on then I must see then times. I am open for suggestions depending upon need.
At the same time the new release of the [Tideland Go Cells](https://github.com/tideland/go-cells) will
be prepared. The architecture has been streamlined and I would like to create a nice example for
the application.


