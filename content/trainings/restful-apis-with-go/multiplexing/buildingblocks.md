+++
date = "2021-11-19T12:00:00+01:00"
draft = false
title = "RESTful APIs with Go - The building blocks together
+++

### Idea

* Register the RESTful API for users

### Example

```go
package main

import (
    "net/http"

    "./pkg/httpx"
    "./pkg/users"
)

func main() {
    mux := http.NewServeMux()
    apimux := httpx.NewNestedMux("/api/v1")

    apimux.Handle("users", httpx.MethodWrapper(users.NewHandler()))
    apimux.Handle("users/addresses", httpx.MethodWrapper(users.NewAddressesHandler()))v
    apimux.Handle("users/contracts", httpx.MethodWrapper(users.NewContractsHandler()))

    // ...

    mux.Handle("/api/v1/", apimux)

    // ...

    http.ListenAndServe(":8080", mux)
}
```

---

[<< NESTING](nesting.md) || [TABLE OF CONTENTS](../index.md)
