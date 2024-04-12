---
title: "Tideland Go Wait v0.2.0"
date: "2022-03-05T22:25:00+01:00"
draft: false
categories: ["development"]
tags: ["golang", "tideland", "library", "release", "wait", "poll", "throttle", "limit"]
---

The package **Tideland Go Wait** reached **v0.2.0** due to a new added feature. It now contains the type `Throttle` to provide a limited processing of events per second, e.g. for web handlers. The events are simple closures or functions with a given signature. The limit and a burst size for the maximum number of events during one call are defined at throttle creation.

### Example

A throttled wrapper of a `http.Handler`.

```go
type ThrottledHandler struct {
    throttle *wait.Throttle
    handler  http.Handler
}

func NewThrottledHandler(limit wait.Limit, handler http.Handler) http.Handler {
    return &ThrottledHandler{
        throttle: wait.NewThrottle(limit, 1),
        handler:  handler,
    }
}

func (h *ThrottledHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    evt := func() error {
        h.ServeHTTP(w, r)
        return nil
    }
    h.throttle.Process(context.Background(), evt)
}
```

The documentation still can be found at [pkg.go.dev](https://pkg.go.dev/tideland.dev/go/wait), the project sources at [GitHub](https://github.com/tideland/go-wait). Enjoy it.

