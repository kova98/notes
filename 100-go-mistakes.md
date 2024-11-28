# 100 Go Mistakes and How to Avoid Them


## 1. Go: Simple to learn but hard to master

Features are missing from go because "they don't fit, affect compilation speed, clarity of design, or because it would make the fundamental system model too difficult.

Simple doesn't mean easy.

Learning from mistakes is very efficient.

"Tell me and I forget. Teach me and I rememer. Involve me and I learn" - love this quote

seven main categories of mistakes
- bugs
  - data races, leaks logic errors, other defects
- needles complexity
  - yagni
- weaker readability
  - uncle bob's clean code reference (lmao)
- suboptimal or unidiomatic organization
  - this should be very valuable, no idea how to organize projects
- lack of api convenience
  - 
- under optimized code
  - not only performance
  - for example, ensure float point operations accuracy
- lack of productivity
  - writing efficient tests (could be interesting)
  - relying on stdlib 
  - profiling tools and linters

## 2. Code and project organization

### 2.1. Unintended variable shadowing

Same as in every other language, variables can be redeclared in inner scopes. However, this is easier to mix
in Go because of the shorthand declaration syntax and multiple returns. 

```go
var client *http.Client
if tracing {
  client, err := createClientWithTracing()
} 

// client is nil here
```

Simple solution would be either declaring the error in the outer scope...

```go
var client *http.Client
var err error
if tracing {
  // both assigned, not declared
  client, err = createClientWithTracing()
} 
```

... or using a temp variable

```go
var client *http.Client
if tracing {
  // both assigned, not declared
  c, err := createClientWithTracing()
  client = c
} 
```

### 2.2. Unnecessary nested code

Return early, reduce nesting, nothing new here. 

### 2.3. Misusing init functions

#### 2.3.1 Concepts

Initially, I thought this was a mistake. This code

```
package main

import "fmt"

var a = func() int {
	fmt.Println("var")
	return 0
}()

func init () {
	fmt.Println("init")
}

func main() {
	fmt.Println("main")
}
```

should output

```
var
init
main
```

but it didn't when I tried it. The reason was I forgot the parentheses at the end, like this:

```
var a = func() int {
	fmt.Println("var")
	return 0
}
```

So it makes sense now. The function is called before the init function because its 
return value (nil) is used within the declaration of a variable.

So, the order of execution in a package is as follows:
1. constants and variable declarations
2. init function
3. main function

If a package depends on another package, like this

```
func main() {
  err := redis.Store("foo", "bar")
}
```

the store package init function will execute before the main init function:
1. constants and variable declarations
2. redis init function
3. main init function
4. main function

If multiple init functions are defined per package, the execution order is based on the source
files' alphabetical order. So, for a package that contains

- package main  
  - a.go  
  - b.go  

the execution is

- a.init()  
- b.init()

Relying on init functions order is a bad idea, as packages can be moved or renamed. 

We can also define multiple init functions within the same source file. In that case, they are 
executed in the source order: first defined, first executed. 

We can also use init functions for side effects with _ imports:

```
package main

import (
  _ "foo"
)
```

This will initialize the foo package and execute its init functions before the main package

Note: init functions cannot be called directly

#### 2.3.2 When to use init functions

**Bad usage example:**

```
var db *sql.DB

func init() {
  d, err := sql.Open("mysql", "whatever")
  if err != nil {
    log.Panic(err)
  }
  err = d.Ping()
  if err != nil {
    log.Panic(err)
  }
  db = d
}
```

- error management is limited, only way to handle is to panic
- if we add tests, the init function is executed before tests
- requires assigning the db connection pool to global variable
  - any function can alter global variables within package
  - unit tests are more complicated

Favor plain functions instead. 

**Good usage example:**

static HTTP config

```
redirect := func(...){
  http.Redirect(...)
}
http.HandleFunc("/blog", redirect)

static := http.FileServer(http.Dir("static"))
http.Handle("/favicon.ico", static)
```

- cannot fail
- no global variables
- does not impact unit tests

### 2.4. Overusing getters and setters

- use sparingly
- idiomatic naming for field value
  - get: Value, set: SetValue

This tip is probably aimed towards java programmers (lol)

### 2.5. Interface pollution

#### 2.5.1. Concepts

Nothing interesting, yagni and isp

#### 2.5.2 When to use interfaces

Nothing interesting here, author explaining abstraction and decoupling and talking about the liskov 
substitution principle, which makes no sense in the context, or any context within a Go code, 
as there is no inheritance. 

The section ends with a bizarre example of "restricting behavior" by defining very fine grained interfaces

```
type intConfig struct {
  ...
}

func (c *IntConfig) Get(value int) {
...
}

func (c *IntConfig) Set(value int) {
  ...
}

type intConfigGetter interface {
  Get() int
}
```

The explanation is that in this particular case, we want to semantically enforce that this 
configuration is read only.

Seems pretty flaky. 



