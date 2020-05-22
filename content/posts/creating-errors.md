---
title: "Creating errors in go"
date: 2018-12-13T23:02:00+05:30
lastmod: 2018-12-13T23:02:00+05:30
tags : ["errors"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

Go is a modern language, but doesn't use try-except blocks to handle errors. Errors are simple values that can be returned from functions and it is common to check for errors before proceeding with your next steps. Let's see the different ways to create your own errors in Go.

Any value is an error if it implements the `error` interface.

```go
    type error interface {
        Error() string
    }
```

<!--more-->

And any value can implement the error interface, by having a `Error` method which returns a string. It's as simple as it gets.

## String based errors

### errors.New

The simplest way to create a custom error is to use the New function of the errors package.

Let's look at the [New function](https://golang.org/src/errors/errors.go?s=293:320#L1) to understand how it is implemented. 

```go
// Package errors implements functions to manipulate errors.
package errors

// New returns an error that formats as the given text.
func New(text string) error {
    return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
    s string
}

func (e *errorString) Error() string {
    return e.s
}
```

This is the entire code for the errors package. We have a errorString struct which has a string field. The `Error()` function of this struct, just returns back the string field s.

Whenever we use New, we create a new variable of errorString struct and return back the address of the variable. 

Now lets look at how we can use errors.New in our own code.

```go
func AreaOfSquare(side float64) (float64, error) {
    if side < 0 {
        return 0, errors.New("Cannot calculate area which has negative side.")
    }
    return side * side, nil
}
```

Now whenever we call AreaOfSquare we will check for the error value and handle it appropriately.

### fmt.Errorf

But let's say we need to format the string that we are returning, we can use the `fmt.Errorf` function to format the string. 
Instead of errors.New, we will have to use the fmt.Errorf.

```go
return 0, fmt.Errorf("Cannot calculate area which has %0.2f as side.", side)
```

## Custom errors with data

String based errors are easy and nice, but if we want the error value to contain even more details about the error that occurec, we should construct our own struct which implements the error interface.

```go
type areaError struct {  
    err    string
    side   float64
}

func (e *areaError) Error() string {  
    return fmt.Sprintf("side %0.2f: %s", e.side, e.err)
}
```

Now, we have our own error type `areaError`, which has a `Error()`. But more important is that it has the field side which stores some extra information about your code. You can inspect this error value and handle the error.

With this change, our AreaOfSquare function is 

```go
func AreaOfSquare(side float64) (float64, error) {
    if side < 0 {
        return 0, &areaError{"side is negative", side}
    }
    return side * side, nil
}
```

We create a new areaError value, with the error string and the side, which gets returned back to the calling function. We have provided the calling function with more information about the error with our custom error struct. 

## Custom errors with data and methods

Let's now write a new package to calculate the area of a rectangle, and we will have it's own areaError struct. There is a slight modification in this struct, as a rectangle can have varying width and height.

```go
type areaError struct {  
    err    string
    width  float64
    height float64
}

func (e *areaError) Error() string {  
    return e.err
}

func (e *areaError) heightNegative() bool {  
    return e.height < 0
}

func (e *areaError) widthNegative() bool {  
    return e.width < 0
}
```

There are two extra methods to the areaError, which return a boolean saying if the height or widgh is negative. These two methods provide more information about the error, whether the area calculation failed because of the height being negative or width being negative. 

Let's now write our AreaOfRect function which uses this areaError struct.

```go
func AreaOfRect(height, width float64) (float64, error) {  
    errString := ""
    if height < 0 {
        errString += "height is less than zero"
    }
    if width < 0 {
        if errString == "" {
            errString = "width is less than zero"
        } else {
            errString += ", width is less than zero"
        }
    }
    if errString != "" {
        return 0, &areaError{errString, height, width}
    }
    return height * width, nil
}
```

In this function, we construct a error string based on which value was negative and use that to create a new areaError value. 

Our main function can use this function, and if it gets back any error, it will use the two methods `heightNegative()` and `widthNegative()` to handle the error correctly.

```go
func main() {  
    height, width := -15.0, -2.0
    area, err := AreaOfRect(height, width)
    if err != nil {
        if err, ok := err.(*areaError); ok {
            if err.heightNegative() {
                fmt.Printf("error: height %0.2f is less than zero\n", err.height)

            }
            if err.widthNegative() {
                fmt.Printf("error: width %0.2f is less than zero\n", err.width)

            }
            return
        }
        fmt.Println(err)
        return
    }
    fmt.Println("area of rect", area)
}
```

## Checking for error type

In this function, `AreaOfRect` returns only `areaError` type and so, we check for the error type by using the `err, ok := err.(*areaError)` line. But if we have a function which can return multiple different error types, we can use a switch-case statement like this.

```go
if err := FunctionToCall(); err != nil {
    switch e := err.(type) {
    case *CustomError1:
        // handle CustomError1
    case *CustomError2:
        // handle CustomError2
    default:
        log.Println(e)
    }
}
```

## Conclusion

This article has just scratched the surface of how error values in go can be used effectively. [Rob Pike's blog post](https://blog.golang.org/errors-are-values) in the official Go Blog walks you through other patterns in handling errors. 