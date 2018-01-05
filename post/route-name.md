+++
title= "route name"
date = "2017-03-11T11:03:42+02:00"
tags= ["route", "name"]
description= '.Name("route") & GetError()'
+++

To be available to know the name of the route within the handler, the `.New` method can be used, example:

```go
router := violetear.New()

router.Handle("*", catchAll, "ALL").New("*")
router.HandleFunc("/post", methodPOST, "POST").New("post")
router.HandleFunc("/get", methodALL, "GET").New("get")
router.HandleFunc("/:any", methodRX).New("rx")
```

To retrieve the name within the handler this could be used:

```go
func theHandler(w http.ResponseWriter, r *http.Request) {
    endpoint := violetear.GetRouteName(r)
	fmt.Fprintf(w, "my name is: %s!", endpoint)
}
```

This is useful when instrumenting your code with tools like [prometheus](/posts/prometheus).


## GetError()

`Handle` and `HandleFunc` return a
[*Trie](https://github.com/nbari/violetear/blob/master/trie.go) this is needed
to be available to set a `Name` per route, therefore to know if an error occurred
when creating the route the method `GetError()` can be used, for example:

```go
router.Handle("*", catchAll, "ALL").New("*")
if err := router.GetError(); err != nil {
    log.Fatal(err)
}
```
