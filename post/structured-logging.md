+++
title= "Structured Logging"
date = "2017-03-11T11:03:40+01:30"
description = "{\"format\": \"json\"}"
tags= ["logging", "json"]
+++

By using a custom logger all the request could be written in a structured format
that can be easily parsed.

Example using JSON format:


```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "os"
    "time"

    "github.com/nbari/violetear"
)

func helloWorld(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("hello world!"))
}

func myLogger(w *violetear.ResponseWriter, r *http.Request) {
    j := map[string]interface{}{
        "Time":        time.Now().UTC().Format(time.RFC3339),
        "RemoteAddr":  r.RemoteAddr,
        "URL":         r.URL.String(),
        "Status":      w.Status(),
        "Size":        w.Size(),
        "RequestTime": w.RequestTime(),
        "RequestID":   w.RequestID(),
    }
    if err := json.NewEncoder(os.Stdout).Encode(j); err != nil {
        log.Println(err)
    }
}

func main() {
    router := violetear.New()
    router.LogRequests = true
    router.Logger = myLogger
    router.Verbose = false
    router.HandleFunc("*", helloWorld)
    log.Fatal(http.ListenAndServe(":8080", router))
}
```

Notice the:

    router.Logger = myLogger

In this example all requests are going to print out to `stdout` similar to this:

```json
{
    "RemoteAddr": "127.0.0.1:51940",
    "RequestID": "",
    "RequestTime": "15.876µs",
    "Size": 12,
    "Status": 200,
    "Time": "2017-10-07T08:36:05Z",
    "URL": "/"
}
```

The `myLogger` function:

    func myLogger(w *violetear.ResponseWriter, r *http.Request) {}

Is using [violetear's Responsewriter](https://github.com/nbari/violetear/blob/master/responsewriter.go),
which contains the following methods:

* Status() the returned HTTP status code
* Size() bytes written
* RequestTime() the time it took the router to dispatch the request
* RequestID() the incoming request-id in case defined
