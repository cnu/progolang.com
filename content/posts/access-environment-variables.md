---
title: "Access Environment Variables in Go"
date: 2018-12-05T23:25:00+05:30
lastmod: 2018-12-05T23:25:00+05:30
tags : ["env vars", "config"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

Environment variables are an easy way to pass configuration settings to your program. This allows you to not store sensitive config settings like database passwords or API keys in your code or local files. This is the way Heroku passes any configuration variables to your application.

To read/write environment variables, Go has three functions in the `os` package.

<!--more-->

## os.Setenv

[os.Setenv](https://golang.org/pkg/os/#Setenv) sets a value of an environment variable by the key name. This is the signature of the function.

```go
func Setenv(key, value string) error
```

## os.Getenv

[os.Getenv](https://golang.org/pkg/os/#Getenv) is the complementary function which allows you to read any environment variable. It takes in a key name and returns back a string. Its upto you to convert into the correct datatype later.

```go
func Getenv(key string) string
```

## os.Unsetenv

[os.Unsetenv](https://golang.org/pkg/os/#Unsetenv) unsets an environment variable with a key name. 

```go
func Unsetenv(key string) error
```

## Example code
Here is a very simple example code which sets an environment variable, prints it by Getting it and then unsets it before exiting.

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    os.Setenv("TMPDIR", "/my/tmp")
    defer os.Unsetenv("TMPDIR")
    fmt.Println(os.Getenv("TMPDIR"))
}
```

## Get back all environment variables

What if you need to get back a slice of all environment variables? There is a handy os.Environ() function which does exactly that.

```go
for _, s := range os.Environ() {
    kv := strings.SplitN(s, "=", 2) // unpacks "key=value"
    fmt.Printf("KEY:%q\tVAL:%q\n", kv[0], kv[1])
}
```

The variable s is a single string which has the key and value split by a `=` character (eg: `EDITOR=/usr/bin/vim`). We use the strings.SplitN function to parse it and store it in kv. Now this makes it easy to check only for a specific list of keys and get the values for those.

