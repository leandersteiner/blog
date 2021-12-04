---
title: "Basic HTTP Clients in Go (net/http)"
date: 2021-12-01
author: "Leander Steiner"
draft: false
description: "Taking a look at http clients in Go using the net/http package"
---

# net/http - Clients

The ```net/http``` package offers simple but powerful abstractions around http tasks.

## Clients

### Creating a Client

Instead of using the default client with ```http.Get()``` we create our own so we can specify our timeout.

{{< code language="go" title="Creating a client" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
client := &http.Client {
    Timeout: 5 * time.Second
}
{{< /code >}}


### Creating a Request

We create a new ```*http.Request``` a pointer to a request object which we will pass to our client for execution.
The context we pass is an empty context that has no timeout. 
We also specify the http method we want to use and could provide a body as the last argument.

{{< code language="go" title="Creating a request" id="2" expand="Show" collapse="Hide" isCollapsed="false" >}}
const url = "https://jsonplaceholder.typicode.com/todos/1"
req, err := http.NewRequestWithContext(context.Background(), http.MethodGet, url, nil)
if err != nil {
	panic(err)
}
{{< /code >}}

### Executing the Request

Now we can execute the request using our client.
We have to make sure that we defer the closing of the responses body which is a ```io.ReadCloser```.

{{< code language="go" title="Executing the request" id="3" expand="Show" collapse="Hide" isCollapsed="false" >}}
res, err := client.Do(req)
if err != nil {
	panic(err)
}
defer res.Body.Close()

if res.StatusCode != http.StatusOK {
	panic("unexpected status")
}
{{< /code >}}

We now have access to a couple of fields on the response object(```*http.Response```)

- ```res.StatusCode``` -> numeric response status
- ```res.Status``` -> textual response status
- ```res.Header``` -> response headers (```type Header map[string][]string```)
- ```res.Body``` -> the ```io.ReadCloser``` to read the response data from

### Reading the Response Data

We can now create a new struct that will hold the response data.
To achieve this we will first create a new anonymous struct that corresponds to the json we receive back as the response.

{{< code language="go" title="Creating a struct to hold the data" id="4" expand="Show" collapse="Hide" isCollapsed="false" >}}
var data struct {
	UserID    int    `json:"userId"`
	ID        int    `json:"id"`
	Title     string `json:"title"`
	Completed bool   `json:"completed"`
}
{{< /code >}}

Now all we have to do is fill the struct with the response data.

{{< code language="go" title="Fill the struct with response data" id="5" expand="Show" collapse="Hide" isCollapsed="false" >}}
err = json.NewDecoder(res.Body).Decode(&data)
if err != nil {
	panic(err)
}

fmt.Printf("%+v\n", data)
{{< /code >}}


Output:
```
{UserID:1 ID:1 Title:delectus aut autem Completed:false}
```

### Full code listing

{{< code language="go" title="Full code" id="6" expand="Show" collapse="Hide" isCollapsed="false" >}}
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"net/http"
	"time"
)

func main() {
	client := &http.Client{
		Timeout: 5 * time.Second,
	}

	const url = "https://jsonplaceholder.typicode.com/todos/1"
	req, err := http.NewRequestWithContext(context.Background(), http.MethodGet, url, nil)
	if err != nil {
		panic(err)
	}

	res, err := client.Do(req)
	if err != nil {
		panic(err)
	}
	defer res.Body.Close()

	if res.StatusCode != http.StatusOK {
		panic("unexpected status")
	}

	var data struct {
		UserID    int    `json:"userId"`
		ID        int    `json:"id"`
		Title     string `json:"title"`
		Completed bool   `json:"completed"`
	}

	err = json.NewDecoder(res.Body).Decode(&data)
	if err != nil {
		panic(err)
	}

	fmt.Printf("%+v\n", data)
}
{{< /code >}}