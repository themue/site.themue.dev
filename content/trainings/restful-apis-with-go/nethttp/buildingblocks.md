+++
date = "2021-11-19T12:00:00+01:00"
draft = false
menu = "trainings"
weight = 101
title = "RESTful APIs with Go - The building blocks together
+++

### Idea

* In-memory Cache Server to store named byte sequences
* HTTP Server to serve the cache

### Example

```go
package main

import (
    "log"
    "net/http"
    "sync"
)

// CacheHandler provides a simple in-memory cache server. Cache is done
// via a map of string to []byte. The sync.RWMutex is used to ensure that
// the cache is thread-safe.
type CacheHandler struct {
    mu    sync.RWMutex
    cache map[string][]byte
}

// newCacheHandler creates the cache server. It's simply needed to create
// the map of string to []byte.
func newCacheHandler() *CacheHandler {
    return &CacheHandler{
        cache: make(map[string][]byte),
    }
}

// ServeHTTP implements the http.Handler interface. It only handles
// the two HTTP methods: GET and POST. The work itself is done by
// private helper methods.
func (h *CacheHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Map HTTP methods to individual methods.
    switch r.Method { 
    case http.MethodGet:
        h.handleGet(w, r)
    case http.MethodPost:
        h.handlePost(w, r)
    default:
        w.WriteHeader(http.StatusMethodNotAllowed)
    }
}

// handleGet handles GET requests. If the path can be found in the cache,
// its data is returned. Otherwise, the response is set to 404. Only
// read lock is needed.
func (h *CacheHandler) handleGet(w http.ResponseWriter, r *http.Request) {
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

// handlePost handles POST requests. It does not matter if the path is known.
func (h *CacheHandler) handlePost(w http.ResponseWriter, r *http.Request) {
    h.mu.Lock()
    defer h.mu.Unlock()

    // Simply write/update the variable.
    h.cache[r.URL.Path] = r.Body
    w.WriteHeader(http.StatusOK)
}

// main runs the cache server.
func main() {
    h := newCacheHandler()
    err := http.ListenAndServe(":8080", h)

    if err != nil {
        log.Fatal(err)
    }
}
```

---

[<< RESPONSES](responses.md) || [TABLE OF CONTENTS](../index.md)
