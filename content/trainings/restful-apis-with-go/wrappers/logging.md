+++
date = "2021-11-19T12:00:00+01:00"
draft = false
title = "RESTful APIs with Go - Logging web requests
+++

### Continue the idea of wrapping

* The interface `http.Handler` allows to wrap any HTTP handler
* This way it's ideal to separate functional and non-functional logic
* While our own handlers care for the business logic, wrappers like `httpx.MethodHandler` and `httpx.NestedMux` care for the HTTP logic
* This can be continued with other aspects of the application, like the logging

### Add a logging handler

```go
package httpx

import (
    "net/http"
)

// Logger defines an interface for many different loggers.
type Logger interface {
    Printf(format string, v ...interface{})
}

// LoggingHandler wraps handlers and logs the requests to them.
type LoggingHandler struct {
    logger  Logger
    handler http.Handler
}

// NewLoggingHandler creates a new logging handler with the given logger and handler.
func NewLoggingHandler(logger Logger, handler http.Handler) *LoggingHandler {
    return &LoggingHandler{
        logger:  logger,
        handler: handler,
    }
}

// ServeHTTP logs the request and calls the wrapped handler.
func (h *LoggingHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    h.logger.Printf("%s %s", r.Method, r.URL.Path)
    h.handler.ServeHTTP(w, r)
}
```

### Example

```go
package main

import (
    "log"
    "net/http"
    "os"

    "./pkg/httpx"
    "./pkg/user"
)

func main() {
    mux := http.NewServeMux()
    apimux := httpx.NewNestedMux("/api/v1")
    apilogger := httpx.NewLoggingHandler(log.New(os.Stdout, "api: ", log.LstdFlags), apimux)

    apimux.Handle("users", httpx.MethodWrapper(user.NewUsersHandler()))
    apimux.Handle("users/addresses", httpx.MethodWrapper(user.NewUsersAddressesHandler()))v
    apimux.Handle("users/contracts", httpx.MethodWrapper(user.NewUsersContractsHandler()))

    // ...

    mux.Handle("/api/v1/", apilogger)

    // ...

    http.ListenAndServe(":8080", mux)
}
```

---

[TABLE OF CONTENTS](../index.md) || [JWT >>](jwt.md)
