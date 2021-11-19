+++
date = "2021-11-19T12:00:00+01:00"
draft = false
menu = "trainings"
weight = 101
title = "RESTful APIs with Go - The net/http package
+++

### Handlers

* The `net/http` package provides a simple, but powerful, HTTP server and client
* It defines the `http.Handler` interface, which is the interface for handlers responding to HTTP requests

```go
type Handler interface {
	ServeHTTP(w ResponseWriter, r *Request)
}
```

* For single functions, it is often easier to use the `http.HandlerFunc` type, a type that implements the `http.Handler` interface

```go
type HandlerFunc func(w ResponseWriter, r *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

* It also provides the `http.Request` struct and `http.ResponseWriter` interface, which represent the HTTP request and response objects

### Servers

* The function `http.ListenAndServe` starts an HTTP server on the given address

```go
func ListenAndServe(addr string, handler Handler) error 
``` 

* Additionally the variant `http.ListenAndServeTLS` starts an HTTPS server on the given address

```go
func ListenAndServeTLS(addr string, certFile string, keyFile string, handler Handler) error
```

### Example

```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

type HelloHandler struct{}

func (h *HelloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello!")
}

func main() {
    err := http.ListenAndServe("localhost:8080", HelloHandler{})

    log.Fatal(err)
}
``` 

---

[TABLE OF CONTENTS](../index.md) || [REQUESTS >>](requests.md)
