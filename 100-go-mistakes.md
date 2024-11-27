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

