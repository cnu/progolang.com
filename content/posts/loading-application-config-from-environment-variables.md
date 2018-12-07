---
title: "Loading Application Config from Environment Variables"
date: 2018-12-07T22:20:00+05:30
lastmod: 2018-12-07T22:20:00+05:30
tags : ["env vars", "config"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

In a previous post, I had written how to read Environment Config. I had also mentioned how you can only store and retrieve string values. But if your application is trying to load config files from it, you would want to load the variables as integer or boolean values. 

When I was new to go, I was using the [strconv](https://golang.org/pkg/strconv/) package to convert the string to various data types. Though it works, it isn't a clean way to write your code. 

Later, I found the [envconfig](https://godoc.org/github.com/kelseyhightower/envconfig) package which loads a config object from environment variables, and my life was changed. The way to use envconfig is really simple. 
<!--more-->
You create a config structure with all the variables you want to load and create a variable of that struct. Then use the envconfig.Process function to load the environment variables into the config variable. 

This is the signature of the Process function. 

    func Process(prefix string, spec interface{}) error

The prefix parameter is a string prefix you can use to maintain your environment variables without any namespace collisions with other programs running on that machine.

Here is a sample program which reads a few environment variables, loads them into a config structure having boolean, string and unsigned integer values. 

    package main

    import (
        "fmt"
        "log"

        "github.com/kelseyhightower/envconfig"
    )

    type Config struct {
        Debug       bool
        DatabaseURL string
        Timeout     uint
    }

    func main() {
        var config Config
        err := envconfig.Process("APP", &config)
        if err != nil {
            log.Fatal(err)
        }

        fmt.Println(config.Debug)
        fmt.Println(config.DatabaseURL)
        fmt.Println(config.Timeout)
    }


If you run the program, you would just get back the default zero values for the different struct members. 

    $ go run envsample.go
    false

    0

But if you set the environment variables and run it again, you can get back the values. On a unix system, you can set the environment variables using the export command.

    export APP_DEBUG=true
    export APP_DATABASEURL=localhost:5432
    export APP_TIMEOUT=60

Now running the program again, will give the output you were expecting. 

    $ go run envsample.go
    true
    localhost:5432
    60

This is very useful when you are writing API servers or microservices which need to load a set of config and run in a docker container or PaaS like Heroku. 
