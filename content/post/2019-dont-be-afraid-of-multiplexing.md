+++
date = "2019-03-23T22:50:00+01:00"
draft = false
title = "Don't be afraid of multiplexing"
tags = ["golang", "http"]
categories = ["development"]
+++

Pretty often you read questions about multiplexing in Go web application on Slack, StackOverflow, or Reddit. Sometimes they think about using libraries like `gorilla/mux`, which is a powerful software, and its alternatives. Depending on individual requirements and constraints these may make sense, but for many cases the standard library or own little packages based on the standard library are more than enough. I'll show the idea behind the Go `net/http` package and how to build own solutions based on it.

## Processing requests

The package `net/http` provides a server which distributes incoming HTTP requests to processing functions in goroutines. The simplest variants are the functions `http.ListenAndServe()` for HTTP and `http.ListenAndServeTLS()` for HTTPS. Also there are `http.Serve()` and `http.ServeTLS()` which do work with individual instances of a `net.Listener` and provide more flexibility in its configuration. Third way with a maximum of flexibility is the creation of an own instance of the `http.Server` which is configurable by many parameters.

All three ways have in common that they need an implementation of the `http.Handler` interface. It only defines one method, 

```
ServeHTTP(w http.ResponseWriter, r *http.Request)
```

This method is responsible to analyse the request `r` and write the answer to the writer `w`. Both types provide different fields and methods for this task.

A simple and convenient form of the interface is the `http.HandlerFunc`. It is the simple function type 

```
func(w http.ResponseWriter, r *http.Request)
```

Here the needed method is implemented quite simple by calling the receiver type. It's nice in Go that function types can have methods too. So now a very simple server is done with only a few lines of code.

```
h := func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("Hello, World!"))
}

http.ListenAndServe(":8080", h)
```

The analysis of the path of a request and the distribution to according functions inside the handler is a bit inconvenient. For example this can be done via a `switch` statement. But it's no fun.

```
func (api *ShopAPI) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    switch {
    case strings.HasPrefix(r.URL.Path, "/api/customers/"):
        api.customersServeHTTP(w, r)
    case strings.HasPrefix(r.URL.Path, "/api/orders/"):
        api.ordersServeHTTP(w, r)
    default:
        http.Error(w,
            "cannot handle request", http.StatusNotFound)
    }
}
```

This might be possible in case of very small APIs, but with growing number of paths this approach will become too static and unmaintainable. Additionally the example is deliberately kept simple. But with paths like `/posts/../api/customers/` or `/api/customers` without the trailing slash this way fails. So the path has to be preprocessed before the request can be handled. A more powerful multiplexing is needed.

Thankfully the `http` package of Go provides a handler which acts as multiplexer, the `http.ServeMux`. Here the `ServeHTTP()` method analyses the request path, computes relative parts, and distributes the request to handlers registered by paths. Those may be absolute for individual resources or for all resources inside a path. The used paths may overlap, for example one handler for `/posts/` and one for `/posts/images/`. When a request contains the longer registered path its handler has the higher priority.

```
mux := http.NewServeMux()

mux.Handle("/posts/", NewPostsHandler())
mux.Handle("/posts/images/", NewPostImagesHandler())
mux.Handle("/api/", NewAPIHandler())
mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusNotFound)
})

http.ListenAndServe(":8080", mux)
```

As long as only the simple  `http.ListenAndServe()` is used you don't need an extra instance of the multiplexer. The package contains the global `DefaultServeMux` and the handler can be registered via `http.Handle("/api/", NewAPIHandler())`. When starting the server you only need to pass `nil` as handler, so `http.ListenAndServe(":8080", nil)`.

## But which method?

Beside the path of a request its HTTP method is important too. Here the Go packages provide nothing, a handler has to do evaluate the field `Request.Method` manually. But by creating a small `MethodMux` it's no problem.

```
type MethodMux struct {
    handlers map[string]http.Handler
}

func (mux *MethodMux) Handle(method string, handler http.Handler) {
    mux.handlers[method] = handler
}

func (mux *MethodMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    handler, ok = mux.handlers[r.Method]
    if !ok {
        http.Error(w,
            "cannot handle request", http.StatusMethodNotAllowed)
    }
    handler.ServeHTTP(w, r)
}
```

So in a RESTful API for a shop on base of the `DefaultServeMux` individual handler and handler functions can be registered for individual HTTP methods.

```
customerAPI := NewMethodMux()

customerAPI.Handle(http.MethodPost, NewCustomerCreateHandler())
customerAPI.Handle(http.MethodGet, NewCustomerReadHandler())
customerAPI.Handle(http.MethodPut, NewCustomerUpdateReplaceHandler())
customerAPI.Handle(http.MethodPatch, NewCustomerUpdateModifyHandler())
customerAPI.Handle(http.MethodDelete, NewCustomerDeleteHandler())

http.Handle("/api/customers/", customerAPI)
```

For many APIs a better approach may be the usage of only one handler but with individual Go methods for the individual HTTP methods. Here a wrapper, a number of interfaces, and the _type assertion_ can help. First you need to define a small interface for each HTTP method.

```
type GetHandler interface {
    ServeGet(w http.ResponseWriter, r *http.Request)
}

type PostHandler interface {
    ServePost(w http.ResponseWriter, r *http.Request)
}

// ...
```

And the wrapper evaluates inside its `ServeHTTP()` method the HTTP method and checks if the given handler implements the matching interface.

```
type MethodWrapper struct {
    handler http.Handler
}

func (mw MethodWrapper) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        if h, ok := mw.handler.(GetHandler); ok {
            h.ServeGet(w, r)
            return
        }
    case http.MethodPost:
        if h, ok := mw.handler.(PostHandler); ok {
            h.ServePost(w, r)
            return
        }
    case ...:
        ...
    }
    mw.handler.ServeHTTP(w, r)
}
```

Now the business logic can be distributed to the according methods if those are provided. Otherwise the `ServeHTTP()` method of the handler itself can provide an error message or some kind of default handling. All fields and methods of the handler are shared between all method handle methods (_Why both share the same name, HTTP and Go?_)

```
type shopAPI struct {}

func (api shopAPI) ServeGet(w http.ResponseWriter, r *http.Request) { ... }

func (api shopAPI) ServePost(w http.ResponseWriter, r *http.Request) { ... }

func (api shopAPI) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    http.Error(w,
        "cannot handle shop request", http.StatusMethodNotAllowed)
}
```

This way the deployment will become one line of code.

```
http.Handle("/shop/", NewMethodWrapper(NewShopAPI()))
```

Once implemented the wrapper can be used for all handlers of the application.

## With security

Another requirement for many web applications is the protection against unauthorized access. One well known technology is the [JSON Web Token](https://jwt.io/). It contains via key secured a payload with standard and individual fields, named claims. Those are for example the user ID (`sub` - subject), the time of issuing (`iat` - issued at), the starting time of the token (`nbf` - not before) and the expiration time (`exp`). An application can add own claims, like roles or allowed paths for the individual user. Via different hashing technologies the JWT standard ensures that the payload hasn't been changed.

The token for a user session has to be generated after the authentication and to be returned to the client. The client has to deliver it as header in the form `Authorization: Bearer <Token>`  with each request. On server-side a security wrapper can read this header, check for consistency and validity, and let only pass authorized requests. Thankfully there are several supporting packages for JWT available.

```
type AuthWrapper struct {
    handler http.Handler
    authURL string
    roles   []string
}

func NewAuthWrapper(
    handler http.Handler, 
    authURL string,
    roles ...string) http.Handler {
    return AuthWrapper{
        handler: handler,
        authURL: authURL,
        roles:   roles...,
    }
}

func (aw AuthWrapper) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if !aw.hasValidToken(r) {
        http.Redirect(w, r, aw.authURL, http.StatusUnauthorized)
        return
    }
    if !aw.isAllowed(r) {
        http.Error(w,
            "access not allowed", http.StatusUnauthorized)
        return
    }
    w.handler.ServeHTTP(w, r)
}
```

If a request contains no token or is a delivered one expired a redirect to a URL for the login has to be done. Also an automatic extension if a token expires in a few minutes is a nice feature. There a many possible strategies, most important is the idea of the wrapping as part of the own toolbox.

```
usersAPI := NewAuthWrapper(NewMethodWrapper(NewUsersAPI()), "admin")

http.Handle("/api/users/", usersAPI)
```

With a trivial interface with only one method is this more simple and convenient possible as with complex interfaces and a huge configuration. Another strength of this approach is also the simple isolated testing of the business logic handlers without any infrastructure tasks. Here Go provides a very useful `httptest.Server` as local test server in the package `net/http/httptest`.

## And RESTful APIs?

The `http.ServeMux` is helpful when working with static paths. But delivering paths with flexible parts to the same handlers is no feature. In case of RESTful APIs individual identifiers are parts or the path the HTTP methods control what to do then. Examples are

- `/api/orders` for adding a new order or retrieving a list of orders,
- `/api/orders/{order-id}` for retrieving, updating, or delete an order,
- `/api/orders/{order-id}/items` for a new order item or retrieving a list of items, and
- `/api/orders/{order-id}/items/{item-id}` retrieving, updating, and delete order item.

It would be nice if this could be done with only two handlers. An `OrdersAPI` for the orders and an `OrdersItemsAPI` for the order items. How can this be done elegantly?

With the `ServeMux` all requests prefixed `/api/orders/` can be assigned to one handler. This one has to process requests around orders and order items. A little helper function for the checking and retrieval of the order ID and the item index should be no problem. So the order ID can retrieved via `id, ok := GetPathElem(r, 3)` very easily. Now only the question on how to distribute the request to the two handlers can be done is left. Once again a simple wrapper helps.

```
type NestedWrapper struct {
    firstHandler  http.Handler
    secondHandler http.Handler
}

func (w NestedWrapper) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    switch len(strings.Split(r.URL.Path, "/")[1:]) {
    case 2, 3:
        w.firstHandler.ServeHTTP(w, r)
    case 4, 5:
        w.secondHandler.ServeHTTP(w, r)
    default:
        http.Error(w,
            "invalid URL", http.StatusNotFound)
    }
}
```

The handlers managed by this wrapper themselves can be wrapped by or use other wrappers, depending on their needs. Those can be the ones for the authorization, for the dispatching of the HTTP methods, or others.

```
ordersAPI := NewAuthWrapper(
    NewNestedWrapper(
        NewMethodWrapper(NewOrdersAPI()), 
        NewMethodWrapper(NewOrdersItemsAPI()),
    ), 
    "dispatcher",
)

http.Handle("/api/orders/", ordersAPI)
```

Surely there are useful applications where the pros of using an existing library is bigger than the cons through complexity and dependencies. But this decision has to be evaluated very thoroughly. Go is designed for simplicity and composition, the basic ideas of Unix. And the package `net/http` shows with a simple interface and the both types for request and response how own flexible toolboxes can easily be created with this philosophy.
