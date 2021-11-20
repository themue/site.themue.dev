+++
date = "2021-11-19T12:00:00+01:00"
draft = false
title = "RESTful APIs with Go - Nesting of handlers
+++

### Requirements in RESTful APIs

* Requests like `GET /api/v1/users/1/addresses/5` show the usage of resources and subresources together with their individual identifiers
* A typical pattern is `{prefix}/{resource}/{id}/{subresource}/{subresource-id}/...`
* The type `http.ServeMux` does not handle a direct matching as names as the part of the path are changing
* So own path helpers and a multiplexer are needed

### Multiplexing of requests path to handlers based on resource names

* `NestedMux` is a multiplexer of handlers based on resource names
* A path is a number of resource names separated by slashes
* E.g. `customers`, `customers/addresses`, and `customers/contracts` are all valid paths

```go
// NestedMux allows to nest handler following the pattern
// {prefix}/{resource}/{id}/{subresource}/{subresource-id}/...
type NestedMux struct {
	mu       sync.RWMutex
	prefix   string
	handlers map[string]http.Handler
}

func NewNestedMux(prefix string) *NestedMux {
	return &NestedMux{
		prefix:   prefix,
		handlers: make(map[string]http.Handler),
	}
}

func (mux *NestedMux) Handle(path string, h http.Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

	mux.handlers[path] = h
}

// ServeHTTP implements http.Handler.
func (mux *NestedMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	mux.mu.RLock()
	defer mux.mu.RUnlock()

	ress := PathToResources(r, mux.prefix)
	path := ress.path()
	h, exists := mux.handlers[path]

	if !exists {
		h = http.NotFoundHandler()
	}

	h.ServeHTTP(w, r)
}
```

---

[<< MULTIPLEXING](multiplexing.md) ||  [TABLE OF CONTENTS](../index.md) || [BUILDING BLOCKS TOGETHER >>](buildingblocks.md)
