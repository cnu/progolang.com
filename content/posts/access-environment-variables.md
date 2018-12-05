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

## os.Setenv

[os.Setenv](https://golang.org/pkg/os/#Setenv) sets a value of an environment variable by the key name. This is the signature of the function.

    func Setenv(key, value string) error

## os.Getenv

[os.Getenv](https://golang.org/pkg/os/#Getenv) is the complementary function which allows you to read any environment variable. It takes in a key name and returns back a string. Its upto you to convert into the correct datatype later.

    func Getenv(key string) string

## os.Unsetenv

[os.Unsetenv](https://golang.org/pkg/os/#Unsetenv) unsets an environment variable with a key name. 

    func Unsetenv(key string) error

## Example code
Here is a very simple example code which sets an environment variable, prints it by Getting it and then unsets it before exiting.

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
