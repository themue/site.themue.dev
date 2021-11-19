+++
date = "2021-11-19T12:00:00+01:00"
draft = false
menu = "trainings"
weight = 101
title = "RESTful APIs with Go - Web responses
+++

### http.ResponseWriter

* The type `http.ResponseWriter` is an interface type that represents the response being sent back to the client
* It defines only three methods
    * `Header()`: returns the header map and allows you to set the response header
    * `Write([]byte) (int, error)`: writes the response body
    * `WriteHeader(int)`: sets the response status code
* The method `Write()` ensures that the response also implements the `io.Writer` interface
* So all functions and methods working with `io.Writer` can also be used writing to the `http.ResponseWriter`

### Example

#### Hello, World!

```go
func (h *Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/plain")
    w.Write([]byte("Hello, World!"))
    w.WriteHeader(http.StatusOK)
}
```

#### Accept

```go
func (h *Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    accept := r.Header.Get("Accept")

    w.Header().Set("Content-Type", "text/plain")
    fmt.Fprintf(w, "Your Accept header is: %s", accept)
    w.WriteHeader(http.StatusOK)
}
```

---

[<< REQUESTS](requests.md) || [TABLE OF CONTENTS](../index.md) || [BUILDING BLOCKS TOGETHER >>](buildingblocks.md)
