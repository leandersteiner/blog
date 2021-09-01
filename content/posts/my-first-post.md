---
title: "My First Post"
date: 2021-09-01T08:13:30+02:00
draft: false
hideReadMore: true
description: "\"Hello, friend?\" That's lame. Maybe I should give you a name?"
author: "Leander Steiner"
---

{{< code language="go" title="main.go" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
package main

import (
	"fmt"
	"os"
)

func main() {
	argsWithProg := os.Args
	argsWithoutProg := os.Args[1:]
	args := os.Args[3]

	fmt.Println(argsWithProg)
	fmt.Println(argsWithoutProg)
	fmt.Println(args)
}
{{< /code >}}

# Basics

The very basics of the go programming language.

## Hello World

### Code

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello World")
}
```

### Run

```
$ go build hello-world.go
$ ./hello-world
```

```
$ go run hello-world.go
```

## Command line arguments

### Read & Process arguments

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	argsWithProg := os.Args
	argsWithoutProg := os.Args[1:]
	args := os.Args[3]

	fmt.Println(argsWithProg)
	fmt.Println(argsWithoutProg)
	fmt.Println(args)
}
```

### Sum of two command line arguments

```go
package main

import (
	"fmt"
	"os"
	"strconv"
)

func main() {
	argsWithProg := os.Args

	numA, err := strconv.Atoi(argsWithProg[1])
	if err != nil {
		fmt.Println(err)
		os.Exit(2)
	}

	numB, err := strconv.Atoi(argsWithProg[2])
	if err != nil {
		fmt.Println(err)
		os.Exit(2)
	}

	result := numA + numB
	fmt.Printf("%d + %d = %d\n", numA, numB, result)

}
```

## Packages & Import

### Downloading third party libraries

```
$ go get -v go.mongodb.org/mongo-driver
```

- The library will be downloaded into the GOPATH directory

### Using the library

```go
package main

import (
	myalias "go.mongodb.org/mongo-driver/mongo"
)

func main() {
  ...
	client, err := myalias.NewClient()
  ...
}

```

## Variables, Constants & Enums

### Variables

- Go is a strong statically typed language
- Type of variables must be fixed at compile time

```go
package main

import "fmt"

func main() {
	var a int
	a = 42

	var aa int = 100

	b := -42

	c := "this is a string"

	var d, e string
	d, e = "var d", "var e"

	f, g := true, false

	fmt.Println(a)
	fmt.Println(aa)
	fmt.Println(b)
	fmt.Println(c)
	fmt.Println(d)
	fmt.Println(e)
	fmt.Println(f)
	fmt.Println(g)
}

```

#### Basic types

Type | Description
---: | ---
`bool` | Boolean (true or false)
`string` | String of characters
`int`, `int8`, `int16`, `int32`, `int64` | Signed integers
`uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr` | Unsigned integers
`byte` | Byte, like uint8
`rune` | Unicode code point
`float32`, `float64` | Floating point numbers
`complex64`, `complex128` | Complex numbers

```go
package main

import "fmt"

func main() {
	var aBool bool = true
	var aString string = "yXXXy"
	var aComplex complex64 = 5i
	var aRune = '€'

	fmt.Println(aBool)
	fmt.Println(aString)
	fmt.Println(aComplex)
	fmt.Println(aRune)
	fmt.Printf("%U\n", aRune)
	fmt.Printf("%c\n",aRune)
}
```

Output:

```
true
yXXXy
(0+5i)
8364
U+20AC
€
```

### Constants

```go
package main

import (
	"fmt"
	"reflect"
)

const (
	Pi = 3.14
	Avogadro float32 = 6.022e23
)

func main() {
	const age = 23
	fmt.Println("What is the value of Pi? Pi is", Pi)
	fmt.Println(reflect.TypeOf(Pi))
	fmt.Println("Avogadro's Number value is", Avogadro)
	fmt.Println(reflect.TypeOf(Avogadro))
	fmt.Println(age)
}

```

Output: 

```
What is the value of Pi? Pi is 3.14
float64
Avogadro's Number value is 6.022e+23
float32
23
```

### Enums

```go
package main

import "fmt"

type DayOfTheWeek uint8

const (
	Monday DayOfTheWeek = iota
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday
	Sunday
)

func main() {
	fmt.Printf("Monday is %d\n", Monday)
	fmt.Printf("Wednesday is %d\n", Wednesday)
	fmt.Printf("Friday is %d\n", Friday)
}

```

Output:

```
Monday is 0
Wednesday is 2
Friday is 4
```

- The iota keyword represents successive integer constants
- It resets to 0 whenever the word `const` appears in the source code

## Functions

- Functions encapsulate a piece of code that performs certain operations
- Function can be called at any point in the code
- A function is the most basic solution to code reuse
- Functions receive none or several parameters and return none or **several** values
- Functions are defined by the `func` keyword, the arguments with their types and the return types

```go
package main

import "fmt"

func main() {
	result := sum(4, 3)
	fmt.Println(result)
	sum, sub := ops(2, 2)
	fmt.Println("2 + 2 =", sum, "\n2 - 2 =", sub)
	b, _ := ops(10, 2)
	fmt.Println("10 + 2 =", b)
}

func sum(a int, b int) int {
	return a + b
}

func ops(a int, b int) (int, int) {
	return a + b, a - b
}
```

Output:

```
7
2 + 2 = 4 
2 - 2 = 0
10 + 2 = 12
```

### Variable numbers of arguments

```go
package main

import "fmt"

func main() {
	total := sum(1, 2, 3, 4, 5)
	fmt.Println(total)
}

func sum(nums ...int) int {
	total := 0
	for _, a := range nums {
		total += a
	}
	return total
}
```

Output:

```
15
```

### Functions as arguments

```go
package main

import "fmt"

func main() {
	c := doit(sum, 2, 3)
	fmt.Println("2 + 3 =", c)
	d := doit(multiply, 2, 3)
	fmt.Println("2 * 3 =", d)
}

func doit(operator func(int, int) int, a int, b int) int {
	return operator(a, b)
}

func sum(a int, b int) int {
	return a + b
}

func multiply(a int, b int) int {
	return a * b
}
```

Output:

```
2 + 3 = 5
2 * 3 = 6
```

### Function closures

- go permits anonymous functions
- these functions can be closures
- closures are functions that can refer to variables outside its body

```go
package main

import "fmt"

func main() {
	a := accumulator(1)
	b := accumulator(2)

	fmt.Println("a", "b")
	for i := 0; i < 5; i++ {
		fmt.Println(a(), b())
	}
}

func accumulator(increment int) func() int {
	i := 0
	return func() int {
		i = i + increment
		return i
	}
}
```

Output:

```
a b
1 2
2 4
3 6
4 8
5 10
```

## Pointers

```go
package main

import "fmt"

func main() {
	x := 100

	a(x)
	fmt.Println("after a x =", x)
	b(&x)
	fmt.Println("after b x =", x)

	fmt.Println("Address of x:", &x)
}

func a (i int) {
	i = 0
}

func b (i *int) {
	*i = 0
}
```

Output:

```
after a x = 100
after b x = 0
Address of x: 0xc00000a0a8
```

## Nil and Zero values

- When variables are declared but not initialized go automatically assigns a default null value for its type
- the keyword `nil` specifies a particular value for every non-initialized type

```go
package main

import "fmt"

func main() {
	var a int
	fmt.Println(a)

	var b *int
	fmt.Println(b)

	var c bool
	fmt.Println(c)

	var d func()
	fmt.Println(d)

	var e string
	fmt.Printf("[%s]", e)
}
```

Output:

```
0
<nil>
false
<nil>
[]
```

## Loops and Branches

### If/Else

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	x := rand.Float32()

	if x < 0.5 {
		fmt.Println("head")
	} else {
		fmt.Println("tail")
	}
}
```

### Switch

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	fingerExample()
	noVariableExample()
}

func fingerExample() {
	var finger int = 1

	switch finger {
	case 0:
		fmt.Println("Thumb")
	case 1:
		fmt.Println("Index")
	case 2:
		fmt.Println("Middle")
	case 3:
		fmt.Println("Ring")
	case 4:
		fmt.Println("Pinkie")
	default:
		fmt.Println("Humans usually have no more than 5 fingers")
	}
}

func noVariableExample() {
	rand.Seed(time.Now().UnixNano())
	x := rand.Float32()

	switch {
	case x < 0.25:
		fmt.Println("Q1")
	case x < 0.5:
		fmt.Println("Q2")
	case x < 0.75:
		fmt.Println("Q3")
	default:
		fmt.Println("Q4")
	}
}
```

One statement for multiple cases:

```go
switch {
case 0:
case 1:
  //statement for 0 and 1
default:
  //default statement
}
```

### For Loops

```go
package main

import "fmt"

func main() {
	x := 5

	counter := x

	for counter > 0 {
		fmt.Println(counter)
		counter--
	}

	for i := 0; i < x; i++ {
		fmt.Println(i)
	}

	fmt.Println()

	for {
		if x % 2 != 0 {
			fmt.Printf("%d is odd\n", x)
			x++
			continue
		}
		break
	}

	for {
		fmt.Println("Endless Loop")
		break
	}
}
```

Output:

```
5
4
3
2
1
0
1
2
3
4

5 is odd
Endless Loop
```

## Errors

```go
package main

import (
	"errors"
	"fmt"
	"math/rand"
	"time"
)

var Musketeers = []string{
	"Athos", "Porthos", "Aramis", "D'Artagnan",
}

func GetMusketer(id int) (string, error) {
	if id < 0 || id >= len(Musketeers) {
		return "", errors.New(fmt.Sprintf("Invalid id [%d]", id))
	}
	return Musketeers[id], nil
}

func main() {
	rand.Seed(time.Now().UnixNano())
	id := rand.Int() % 6

	musketeer, err := GetMusketer(id)
	if err == nil {
		fmt.Printf("[%d] %s", id, musketeer)
	} else {
		fmt.Println(err)
	}
}
```

## Defer

- defer function to be executed after encompassing function ends 
- functions are pushed onto a stack
- they are popped when encompassing function ends -> reverse order

```go
package main

import "fmt"

func main() {
	defer CloseMsg()

	fmt.Println("Doing something...")
	defer fmt.Println("Certainly Closed!!!")
	fmt.Println("Doing something else...")
}

func CloseMsg() {
	fmt.Println("Closed!!!")
}
```

Output:

```
Doing something...
Doing something else...
Certainly Closed!!!
Closed!!!
```