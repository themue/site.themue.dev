+++
date = "2021-11-19T12:00:00+01:00"
draft = false
title = "RESTful APIs with Go - Handling HTTP methods
+++

### HTTP methods are verbs

* HTTP requests contain different methods, such as GET, POST, PUT, DELETE, etc.
* The methods are used to indicate the desired action to be performed on the resource
* So one task is to analyse the methods and decide which one is the most suitable for your application
* We've seen how to do a `switch` statement to handle the methods and to handle

```go
switch r.Method {
case http.MethodGet:
    // Handle GET.
    h.handleGet(w, r)
case http.MethodPost:
    // Handle POST.
    h.handlePost(w, r)
default:
    // Handle other methods or return error.
    ...
}
```

### But why repeat this for every handler?

* Let's create our own helper package and use the power of interfaces to handle the methods
* So we first create our HTTP helper package, named e.g. `pkg/httpx`

```go
// Package httpx contains helper functions for the daily work with HTTP.
package httpx
```

* And now create our method wrapper `pkg/httpx/methods.go`
* It defines individual interfaces for each HTTP method
* The wrapper contains a `http.Handler` interface
* Its `ServeHTTP` distributes the requests to the handler methods based on a type switch

```go
package httpx

import (
    "net/http"
)

type GetHandler interface {
    ServeHTTPGet(w http.ResponseWriter, r *http.Request)
}

type PostHandler interface {
    ServeHTTPPost(w http.ResponseWriter, r *http.Request)
}

type PutHandler interface {
    ServeHTTPPut(w http.ResponseWriter, r *http.Request)
}

type DeleteHandler interface {
    ServeHTTPDelete(w http.ResponseWriter, r *http.Request)
}

// ...

// MethodHandler wraps a http.Handler implementing also individual httpx handler
// interface. It distributes the requests to the handler methods based on a type
// switch In case of no matching method a http.ErrMethodNotAllowed is returned.
type MethodHandler struct {
    handler http.Handler
}

func NewMethodHandler(h http.Handler) *MethodHandler {
    return &MethodHandler{
        handler: h,
    }
}

// ServeHTTP implements the http.Handler interface.
func (h *MethodHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	notAllowed := func() {
		errtxt := http.StatusText(http.StatusMethodNotAllowed) + ": " + r.Method
		http.Error(w, errtxt, http.StatusMethodNotAllowed)
	}
	switch r.Method {
	case http.MethodGet:
		if hh, ok := h.handler.(GetHandler); ok {
			hh.ServeHTTPGet(w, r)
			return
		}
		notAllowed()
	case http.MethodPost:
		if hh, ok := h.handler.(PostHandler); ok {
			hh.ServeHTTPPost(w, r)
			return
		}
		notAllowed()
	case http.MethodPut:
		if hh, ok := h.handler.(PutHandler); ok {
			hh.ServeHTTPPut(w, r)
			return
		}
		notAllowed()
	case http.MethodDelete:
		if hh, ok := h.handler.(DeleteHandler); ok {
			hh.ServeHTTPDelete(w, r)
			return
		}
		notAllowed()
    // ...
    default:
        // Fall back to default for any other method.
        h.h.ServeHTTP(w, r)
    }
}
```

### Example

```go
package main

import (
    "log"
    "net/http"
    "sync"

    "./pkg/httpx"
)

// CacheHandler provides a simple in-memory cache server. Cache is done
// via a map of string to []byte. The sync.RWMutex is used to ensure that
// the cache is thread-safe.
type CacheHandler struct {
    mu    sync.RWMutex
    cache map[string][]byte
}

// NewCacheHandler creates the cache server. It's simply needed to create
// the map of string to []byte.
func newCacheHandler() *CacheHandler {
    return &CacheHandler{
        cache: make(map[string][]byte),
    }
}

// ServeHTTPGet handles GET requests. If the path can be found in the cache,
// its data is returned. Otherwise, the response is set to 404. Only
// read lock is needed.
func (h *CacheHandler) ServeHTTPGet(w http.ResponseWriter, r *http.Request) {
    h.mu.RLock()
    defer h.mu.RUnlock()

    // Check if path is known.
    data, ok := h.cache[r.URL.Path]
    if !ok {
        w.WriteHeader(http.StatusNotFound)
        return
    }
    w.Write(data)
    w.WriteHeader(http.StatusOK)
}

// ...

// main runs the cache server.
func main() {
    h := httpx.NewMethodHandler(newCacheHandler()) // Use the method handler as wrapper.
    err := http.ListenAndServe(":8080", h)

    if err != nil {
        log.Fatal(err)
    }
}
```

---

[TABLE OF CONTENTS](../README.md) || [CRUD >>](crud.md)
