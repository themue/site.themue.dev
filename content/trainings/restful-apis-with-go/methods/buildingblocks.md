+++
date = "2021-11-19T12:00:00+01:00"
draft = false
title = "RESTful APIs with Go - The building blocks together
+++

### Idea

* The cache server will be changed to a JSON cache server
* Each entry contains an ID and a raw content

### Example

```go
package main

import (
    "log"
    "net/http"
    "sync"

    "./pkg/httpx"
)

// JSONDoc describes one entry in the cache server.
type JSONDoc struct {
    ID      string          `json:"id"`
    Content json.RawMessage `json:"content"`
}

// JSONCacheHandler provides a simple JSON in-memory cache server. Cache is
// done via a map of string to JSONDoc. The JSONDoc contains the ID and a
// raw content. The sync.RWMutex is used to ensure that the cache is
// thread-safe.
type JSONCacheHandler struct {
    mu    sync.RWMutex
    cache map[string]JSONDoc
}

// NewJSONCacheHandler creates the cache server. It's simply needed to create
// the map of string to JSONDoc.
func NewJSONCacheHandler() {
    return &JSONCacheHandler{
        cache: make(map[string]JSONDoc),
    }
}

// ServeHTTPGet retrieves a JSO document out of cache.
func (h *JSONCacheHandler) ServeHTTPGet(w http.ResponseWriter, r *http.Request) {
    h.mu.RLock()
    defer h.mu.RUnlock()

    ids := httpx.ParseResourceIDs(r, "/api/v1")
    if ids == nil {
        http.Error(w, "resource and ID are missing", http.StatusInvalidRequest)
        return
    }
    if ids[0].Name != "json-cache" {
        http.Error(w, "resource is not json-cache", http.StatusBadRequest)
        return
    }
    id := ids[0].ID
    doc, ok := h.cache[id]
    if !ok {
        http.Error(w, "JSON document not found", http.StatusNotFound)
        return
    }
    err := https.WriteBody(w, httpx.ContentTypeJSON, doc)
    if err != nil {
        log.Printf("Error writing body: %v", err)
    }
}

// ServeHTTPPost adds a new JSON document to the cache.
func (h *JSONnewCacheServer) ServeHTTPPost(w http.ResponseWriter, r *http.Request) {
    h.mu.Lock()
    defer h.mu.Unlock()

    ids := httpx.ParseResourceIDs(r, "/api/v1")
    if ids == nil {
        http.Error(w, "resource is missing", http.StatusInvalidRequest)
        return
    }
    if ids[0].Name != "json-cache" {
        http.Error(w, "resource is not json-cache", http.StatusBadRequest)
        return
    }
    var doc JSONDoc
    err := httpx.ReadBody(r, &doc)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    _, ok := h.cache[doc.ID]
    if ok {
        http.Error(w, "JSON document already exists", http.StatusConflict)
        return
    }
    h.cache[doc.ID] = doc
    w.WriteHeader(http.StatusCreated)
}

// ...

// main runs the cache server.
func main() {
    cs := httpx.NewMethodHandler(NewJSONCacheHandler()) // Use the wrapper.
    err := http.ListenAndServe(":8080", cs)

    if err != nil {
        log.Fatal(err)
    }
}
```

---

[<< JSON](json.md) || [TABLE OF CONTENTS](../index.md)
