---
title: "Basic HTTP Servers in Go (net/http)"
date: 2021-12-04
author: "Leander Steiner"
draft: false
description: "Taking a look at http servers in Go using the net/http package"
---

# net/http - Servers

The ```net/http``` package offers simple but powerful abstractions around http tasks.

## Servers

### Creating a Server

Instead of using the default server with ```http.HandleFunc(string, func(http.ResponseWriter, *http.Request))``` we create our own so we can specify our timeout.

{{< code language="go" title="Creating a server" id="server" expand="Show" collapse="Hide" isCollapsed="false" >}}
s := http.Server{
		Addr:         ":8080",
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 15 * time.Second,
		IdleTimeout:  30 * time.Second,
		Handler:      nil, // handler to invoke, http.DefaultServeMux if nil
	}
{{< /code >}}

## Handlers

### Creating a Handler

Now that we have a Server, we want it to respond to incoming HTTP requests. We already saw that we can provide a handler when creating a new server. You can give it any type that implements the ```http.Handler``` interface.

{{< code language="go" title="The Handler interface" id="handler" expand="Show" collapse="Hide" isCollapsed="false" >}}
type Handler interface {
	ServeHTTP(http.ResponseWriter, *http.Request)
}
{{< /code >}}

We can easily create such a type like this:

{{< code language="go" title="Creating a type that satisfies the http.Handler interface" id="custom-handler-type" expand="Show" collapse="Hide" isCollapsed="false" >}}
type HealthHandler struct{}

func (hh HealthHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Server OK\n"))
}
{{< /code >}}

Now that we have a handler we can set it as the default handler for our server so we can test out that everything works.

{{< code language="go" title="Creating a server that responds to requests" id="server-with-handler" expand="Show" collapse="Hide" isCollapsed="false" >}}
package main

import (
	"net/http"
	"time"
)

type HealthHandler struct{}

func (hh HealthHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Server OK\n"))
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/health", HealthHandler{})
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Welcome to the Website\n"))
	})

	s := http.Server{
		Addr:         ":8080",
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 15 * time.Second,
		IdleTimeout:  30 * time.Second,
		Handler:      mux,
	}

	err := s.ListenAndServe()
	if err != nil {
		if err != http.ErrServerClosed {
			panic(err)
		}
	}
}
}
{{< /code >}}

If we now send an HTTP request to ```127.0.0.1:8080``` we will get the response ```Server OK``` as plain text.
To test this out we can use curl to send the request:

```bash
curl 127.0.0.1:8080
```

Result:
```
Server OK
```

## Routing

### Built-in HTTP Request Multiplexer ```http.ServeMux```

For routing, go offers its built-in multiplexer ```http.ServeMux``` which implements the ```http.Handler``` interface.
Whenever we use ```http.Handle()``` or ```http.HandleFunc()``` without creating our own ```http.ServeMux```, go registers our routes on [a default ```http.ServeMux```](https://cs.opensource.google/go/go/+/refs/tags/go1.17.3:src/net/http/server.go;l=2248;bpv=0;bpt=1).

To create our own ```http.ServeMux``` we can use the factory function ```http.NewServeMux```. Once we have an ```http.ServeMux``` we can start to register routes on it. It offers two methods to accomplish this:

- ```Handle(pattern string, handler http.Handler)```
- ```HandleFunc(pattern string, handler func(http.ResponseWriter, *http.Request))```

The first one expects an ```http.Handler``` the second allows us to pass a function that has the same signature as an ```http.Handler```.
Let's look at how we would use these two methods to create a server that responds to two different routes.

{{< code language="go" title="Creating a server that responds to different routes" id="server-with-routes" expand="Show" collapse="Hide" isCollapsed="false" >}}
package main

import (
	"net/http"
	"time"
)

type HealthHandler struct{}

func (hh HealthHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Server OK\n"))
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/health", HealthHandler{})
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Welcome to the Website\n"))
	})

	s := http.Server{
		Addr:         ":8080",
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 15 * time.Second,
		IdleTimeout:  30 * time.Second,
		Handler:      mux,
	}
	err := s.ListenAndServe()
	if err != nil {
		if err != http.ErrServerClosed {
			panic(err)
		}
	}
}
{{< /code >}}

We can now set the handler on our server to the ```http.ServeMux``` we created, since it implements the ```http.Handler``` interface.
Now all requests go through our mux which routes our registered handlers to the requested url.

### Packages for routing

The default ```http.ServeMux``` lacks a lot of functionality we came to expect from routers.
The below list shows some packages dedicated to making routing as easy as possible.

- [gorilla/mux](https://github.com/gorilla/mux)
- [julienschmidt/httprouter](https://github.com/julienschmidt/httprouter)
- [go-chi/chi](https://github.com/go-chi/chi)
- [bmizerany/pat](https://github.com/bmizerany/pat)