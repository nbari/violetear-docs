+++
title= "Websocket Hijack"
date = "2017-03-11T11:03:42+02:00"
description = "Hijack, Flusher"
tags= ["Hijack", "websocket"]
+++


To use all the available interfaces in the [net/http](https://golang.org/pkg/net/http/) packages, disable the `LogRequests` option:

    router := violetear.New()
    router.LogRequests = false


The reasons for this is that when using `LogRequests` a custom
[responseWriter](https://github.com/nbari/violetear/blob/master/responsewriter.go) is used and it doesn't extend to all available interfaces in the `net/http`
package.
