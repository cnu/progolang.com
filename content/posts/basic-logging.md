---
title: "Basic Logging in Go"
date: 2018-12-09T23:45:00+05:30
lastmod: 2018-12-09T23:45:00+05:30
tags : ["logging", "errors"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

Any application you write, should have proper logging to ensure you are able to monitor and identify problems in production. You can't and shouldn't run a debugger in a production environment and logs are the only proper way to get an idea of what is happening in your code. 

Go has a native logging package called `log`. It has a standard Logger, which has helper functions to log messages and errors. The default Logger prints the date and time stamp of the logged error, which makes it much more useful than using `fmt.Println`

<!--more-->

## Basic Logging

Lets see a quick example of how to log when you get an error.

    package main

    import (
        "errors"
        "fmt"
        "log"
    )

    func div(a, b float64) (ret float64, err error) {
        if b == 0 {
            return 0.0, errors.New("can't divide by 0")
        }
        return a / b, nil
    }

    func main() {
        ret, err := div(1.0, 0.0)
        if err != nil {
            log.Fatal(err)
        }
        fmt.Println(ret)
    }

We have a simple function which divides a number by another. But if the second number is 0, we return an error value saying, we can't divide by 0. 

In the main function, we check for this error and use the `log.Fatal(err)` function call to display the error and exit out. log.Fatal always calls os.Exit(1) after printing the error message. 

Running this program prints this error message:

`2018/12/09 22:48:51 can't divide by 0`

# Logging to a file

By default the standard logger, logs to standard error. But if you want to log to a different file instead, you would have to use the `log.SetOutput(f)` function. SetOutput takes in any value which satisfies the `io.Writer` Interface. Lets see a simple example which logs to a local file.

    package main
    import (
            "log"
            "os"
    )

    func main() {
            f, err := os.OpenFile("app.log", os.O_WRONLY|os.O_CREATE|os.O_APPEND, 0644)
            if err != nil {
                    log.Fatal(err)
            }
            defer f.Close()
            log.SetOutput(f)
            log.Println("Application started")
            // Do the rest 
    }

We open a file called app.log in write only, append mode and create a new file if it doesn't exist. Then we use log.SetOutput to set this as the logging output file. 
After this, whenever we use log.Print, log.Fatal, log.Panic functions, it will push them to the app.log file. 

This is a quick and easy way to setup logging in your go code. Takes the same number of characters as using a `fmt.Println`, but with the log package, you get much more control over where and how the output is stored.