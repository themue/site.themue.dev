+++
date = "2019-05-16T23:01:00+01:00"
draft = false
title = "Enjoy functions"
tags = ["golang", "functions"]
categories = ["development"]
+++

What shall be special when working with functions? They are part of software development almost since beginning, there's even the paradigm of functional programming. So what's special with functions in Go? Simple answer: nothing. But true to the motto of the language the work with functions and their relatives, the methods, is very pragmatic. And so they get parts of elegant solutions.

Let's start with quite simple functions. They can be small, having none, one, more or a variable number of parameters. Almost the same applies for the return values. There can be none, one, or more. But no variable number. In this case you would have to return a slice of the wanted type.

```
func VerySimple() {
    ...
}

func TotalLen(v ...string) (int, error) {
    ...
}
```

While the first function is simple and closed, without parameters and return values, the second one takes none to an unlimited number of `string` values as parameters. It shall sum up all lengths of all strings. But it also shall return an error in case of an empty string. So here will be returned two values – the counted length and a possible error which may be `nil`. The convention to return errors as single or last values can be found all over the standard library.

These two function aren't surprising yet. But it's nice that functions, or to become more precise the signatures of functions, are _only_ types. So we easily can declare an own function type.

```
type FilterFunc func(string) bool
```

And where we use this type its implementation can be exchanged easily.

```
func Filter(in []string, isOK FilterFunc) []string {
    var out []string
    for _, s := range in {
        if isOK(s) {
            out = append(out, s)
        }
    }
    return out
}
```

Now when calling `Filter()` we can pass individual filter functions. As example when wanting to filter out all strings longer than 20 characters it's quite simple.

```
out := Filter(in, func(s string) bool {
    return len(s) <= 20
})
```

In this case an anonymous and locally defined function is passed. Thankfully Go is very flexible here. It only cares for the signature of the function. So even regular functions defined in the same or a different package can be passed as argument.

```
// MaxTwenty in package foo.
func MaxTwenty(s string) bool {
    return len(s) <= 20
}

// Filter in package bar.
out := Filter(in, foo.MaxTwenty)
```

As you can see we only pass the identifier of the function and don't call it. A direct call would need the argument and return its result as argument for `Filter()`, which won't compile. By only passing the identifier the filter instead knows which function it shall user for `isOK`. The compiler passes only the reference to the function.

But functions may not only be arguments, they also can be returned by functions. What's nice here is that the returned function has access to all higher-level variables. These may be the parameters of the constructing functions or calculated variables inside of it. This way closures are built.

```
func MakeLenFilter(max int) FilterFunc {
    return func(s string) bool {
        return len(s) <= max
    }
}

. . .

out := Filter(in, MakeLenFilter(80))
```

So we now've got a function that creates and returns an anonymous function and a function that receives this function as argument. This type of higher-order functions is well known since a long time. And Go has integrated them in a simple and pragmatic way, with no special syntax as many other languages are doing it.

As mentioned above methods are nothing else than functions. Their receiver is only a different kind of reference and they have full access to it. In case of a structure they can use all fields as parameters for their work. But it don't always have to be complex structures. Sometimes a simple value is enough, as in the example above the length of the strings to filter would be a good one. So we simply can define a type derived from the builtin `int`. This now gets a method with the same signature as the filter function. That now can use the integer value as parameter for its work which makes the filter flexible.

```
type LengthFilter int

func (lf LengthFilter) IsOK(s string) bool {
    return len(s) <= lf
}

. . .

lf := LengthFilter(50)
out := Filter(in, lf.IsOK)
```

## Some examples
The shown `Filter()` function for a set of data, here a slice of strings, is a well known member of a family of functions with functions as parameter. This family also contains `Map()` and `Aggregate()`. The first one for example could expect a `func(string) string` for the conversion of all input strings into upper or lower cases. Others could cut or pad for a unique length.

```
type MapFunc func(string) string

func Map(in []string, update MapFunc) []string {
    var out []string
    for _, s := range in {
        out = append(out, update(s))
   }
    return out
}

. . .

out := Map(in, func(s string) string {
    return strings.ToUpper(s)
})
```

Input and output of `Filter()` and `Map()` are slices of the same type. So you can nest them easily.

```
comDomains := Filter(
    Map(emails, ExtractDomain),
    MakeDomainFilter("com"),
)
```

Surely you also can create `Map()` functions with different types for input and output. So for example from string to integer to map from strings to lengths or checksums. In case of `Aggregate()` the task is the reduction of a set of values to one.

```
type AggregateFunc func(int, string) int

func Aggregate(
    initial int
    in []string, 
    aggregate AggregateFunc,
) int {
    var out int = initial
    for _, s := range in {
        out = aggregate(out, s)
   }
    return out
}

. . .

length := Aggregate(0, in, func(v, s string) int {
    return v + len(s)
})
```

These kinds of functions for different types and different already provided filter, mapping, and aggregation functions build a useful package. Sadly today it's sometimes a bit inconvenient due to the static typing in Go. You need multiple implementation for the different function signatures. Here dynamic typed languages have an advantage. But with the possibly upcoming generics in Go this may change.

Another nice example is the configuration of structured types. Here often the parameter fields should be private, should have default values, and should mostly only be set during instantiation. So one way is a constructor with all parameters, but here I need an explicit setting. Or I create a number of different constructors which gets horrific in case of an increasing number of different options.

So here we use functions for setting options, functions for the creation of those option functions, and a constructor function taking a variadic number of option functions. Sounds a bit strange? No problem, it will get clearer soon. For example we create a type `Pinger` in the package `pinger`. This type shall ping an IP address inside a goroutine and in case of a number of missing replies it calls a callback.

```
type Callback func(Pinger)

type Pinger struct {
    ip        net.IP
    interval  time.Duration
    tolerance int
    callback  Callback
    shallLog  bool
}
```

For the options we now define the type `Option` as `type Option func(*Pinger)`. Implementations of this one will be passed to the constructor. After the setting of the default values the options will be applied to the created pinger. Afterwards the goroutine will be started and the instance returned.

```
func New(ctx context.Context, options ...Option) *Pinger {
    p := &Pinger{
        ip:        net.IPv4(127, 0, 0, 1),
        interval:  5 * time.Second,
        tolerance: 3,
        callback:  func(cbp Pinger) {
            fmt.Fprintf(
                os.Stderr,
                "pinging %v failed\n", 
                cbp.ip,
            )
        },
        shallLog: true,
    }
    for _, option := range options {
        option(p)
    }
    go p.backend(ctx)
    return p
}
```

So far it's pretty clear, the option functions may modify the pinger. But how do we get those functions? This is the task of different ones with more speaking names based on the pinger parameter. They've got own parameters they validate before setting the value inside the pinger. Incase of groups of parameters the functions my have them all. Or in case of boolean options two functions without parameters may be even better readable.

```
func IP(ip net.IP) Option {
    return func(p *Pinger) {
        if ip != nil {
            p.ip = ip
        }
    }
}

func Interval(i time.Duration) Option {
    return func(p *Pinger) {
        p.interval = i
    }
}

. . .

func NoLogging() Option {
    return func(p *Pinger) {
        p.shallLog = false
    }
}
```

Now the pinger can be created in many ways. Without any option by using the default values, with some of the options, or with all of them. And the more options a type has got the more advantages this way has.

```
p1 := pinger.New(ctx)
p2 := pinger.New(ctx, pinger.IP(net.IPv4(8, 8, 8, 8)))
p3 := pinger.New(
    ctx,
    pinger.IP(net.IPv4(8, 8, 8, 8)),
    pinger.Interval(time.Minute),
    pinger.Tolerance(5),
    pinger.Callback(func(p Pinger) {
        failures <- p
    }),
    pinger.NoLogging(),
)
```

Another wonderful feature is that functions as regular types can be sent via channels too. This can be useful for concurrent types which shall execute only one modifying action at a time. Languages like Erlang/OTP doing this following to the actor model. Here messages are sent into a message box and they will be processed sequentially. Same is simply possible in Go with the processing of functions sent to a loop running inside a goroutine.

```
type MyActor struct {
    a     int
    b     bool
    c     string
    tasks chan func()
}

func New() *MyActor {
    ma := &MyActor{
        tasks: make(chan func(), 1),
    }
    go ma.backend()
    return ma
}

func (ma *MyActor) backend() {
    for task := range tasks {
        task()
    }
}

func (ma *MyActor) do(task func()) {
    wait := make(chan struct{})
    ma.tasks <- func() {
        task()
        close(wait)
    }
    <-wait
}
```

Now the wanted functions with changes or requests of fields can be implemented. Here the method `do()` ensures the synchronization of the calls. It could be even better implemented with checking if the `tasks` channel is already closed or with an optional timeout when waiting for the response.

All here sent tasks will be processed sequentially and concurrent access to fields of the actor will be avoided.

```
func (ma *MyActor) SetA(v int) {
    ma.do(func() {
        ma.a = v   
    })
}

func (ma *MyActor) IsPositive() bool {
    var positive bool
    ma.do(func() {
        positive = ma > 0
    })
    return positive
}
```

There are many more helpful use cases for functions as parameter, as return value or as field of a  `struct`. So easily a state machine can be implemented, where all event processing handler functions do have the same signature and return the handler function for the next event. Depending on the state this could be the same or a different one. When returning `nil` the end of the state machine is reached. 

```
type Event struct { ... }

type EventHandler func(e Event) (EventHandler, error)

type StateMachine struct {
    a      int
    b      bool
    c      string
    handle EventHandler
}

func New() *StateMachine {
    . . .
}

func (sm *StateMachine) Handle(e Event) error {
    if sm.handler == nil {
        return errors.New("...")
    }
    h, err := sm.handle(e)
    if err != nil {
        sm.handle = nil
        return err
    }
    sm.handle = h
    return nil
}
```

Based on this little core you now can implement the wanted event handlers. They evaluate the event, may change the fields of the state machine and return a reference to the following state.

```
func (sm *StateMachine) idle(
    e *Event,
) (EventHandler, error) {
    switch e.Action {
    case "buttonPressed":
        sm.b = true
        return sm.active, nil
    case ...:
        ...
    }
    return sm.idle, nil
}

func (sm *StateMachine) active(
    e *Event,
) (EventHandler, error) {
    ...
}

func (sm *StateMachine) outOfService(
    e *Event,
) (EventHandler, error) {
    ...
}
```

Final variant I want to show is how functions as types can provide methods too. Take a look at the package `net/http`. Here handler for web requests implement the interface `http.Handler`. But sometimes you only want one simple function, no implementation of the interface. Here simply implement the function type `http.HandlerFunc`. It only is a simple function signature like `Handler.ServeHTTP()`. And it implements the interface itself with this method. Here the HTTP function is the receiver and will be called inside `ServerHTTP()`.

```
type HandlerFunc func(w ResponseWriter, r *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

These examples may give you an overview over the power of functions in Go and how seamless they work together with other functions and types. If as normal functions, as types, as parameters, as return values, or as fields of structs, with name or anonymous. They are an important part of the implementation of powerful, flexible, and good maintainable solutions.

## Some links
- [Function types](https://golang.org/ref/spec#Function_types)
- [Function declarations](https://golang.org/ref/spec#Function_declarations)
- [Method declarations](https://golang.org/ref/spec#Function_declarations)
- [Function literals](https://golang.org/ref/spec#Function_literals)
- [Effective Go - Functions](https://golang.org/doc/effective_go.html#functions)
- [http.HandlerFunc](https://golang.org/pkg/net/http/#HandlerFunc)
 