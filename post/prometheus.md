+++
title= "prometheus"
date = "2017-03-11T11:03:41+02:00"
tags= ["prometheus", "metrics"]
description= "/metrics"
+++

Basic [prometheus](prometheus.io) example with [HTTP Basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) for the `/metrics` endpoint:

Notice the use of `.Name("endpoint")` and `GetRouteName`:

    endpoint := violetear.GetRouteName(r)
    c.WithLabelValues(endpoint).Observe(time.Since(start).Seconds())

This helps to create the labels using the defined names when creating the route.

```go
package main

import (
	"crypto/subtle"
	"fmt"
	"log"
	"net/http"
	"time"

	"github.com/nbari/violetear"
	"github.com/nbari/violetear/middleware"
	"github.com/prometheus/client_golang/prometheus"
)

func index(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "index %s!", r.URL.Path)
}
func foo(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "foo %s!", r.URL.Path)
}
func bar(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "bar %s!", r.URL.Path)
}

// BasicAuth secure /metrics endpoint by using the defined username and password
func BasicAuth(next http.Handler, username, password, realm string) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		user, pass, ok := r.BasicAuth()
		if !ok || subtle.ConstantTimeCompare([]byte(user), []byte(username)) != 1 || subtle.ConstantTimeCompare([]byte(pass), []byte(password)) != 1 {
			w.Header().Set("WWW-Authenticate", `Basic realm="`+realm+`"`)
			http.Error(w, "Unauthorized.", http.StatusUnauthorized)
			return
		}
		next.ServeHTTP(w, r)
	})
}

func secondMW(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		log.Println("Second middleware")
		// do something here
		next.ServeHTTP(w, r)
	})
}

func counterMW(c *prometheus.HistogramVec) middleware.Constructor {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			start := time.Now()
			log.Println("Executing counter middleware")
			// do something here
			next.ServeHTTP(w, r)
			log.Println("updating prometheus counters")
			endpoint := violetear.GetRouteName(r)
			c.WithLabelValues(endpoint).Observe(time.Since(start).Seconds())
		})
	}
}

func main() {
	counter := prometheus.NewHistogramVec(
		prometheus.HistogramOpts{
			Namespace: "myAPI",
			Name:      "requests_total",
			Help:      "Total number of requests.",
		}, []string{"endpoint"})
	prometheus.MustRegister(counter)

	// midleware
	stdChain := middleware.New(counterMW(counter), secondMW)

	router := violetear.New()
	router.Handle("/", stdChain.ThenFunc(index)).Name("root")
	router.Handle("/foo", stdChain.ThenFunc(foo), "GET").Name("foo")
	router.Handle("/bar", stdChain.ThenFunc(bar), "POST").Name("bar")
	router.Handle("/metrics", BasicAuth(prometheus.Handler(),
		"user",
		"password",
		"Please enter username and password"),
	)

	// configure server
	srv := &http.Server{
		Addr:           ":8080",
		Handler:        router,
		ReadTimeout:    5 * time.Second,
		WriteTimeout:   7 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	log.Fatal(srv.ListenAndServe())
}
```

gist: https://gist.github.com/nbari/04ce9bf4080e52951aedd1fced0a16bd
