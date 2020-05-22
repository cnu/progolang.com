---
title: "Waiting for Goroutines to complete"
date: 2018-12-15T22:37:00+05:30
lastmod: 2018-12-15T22:37:00+05:30
tags : ["goroutines"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

Goroutines are one of the best features of Go language. 
It is very easy to run a function as a goroutine, you just have to use the keyword go before the function call. 

Here is a very simple function which sleeps for `n` seconds and prints a line.

<!--more-->

```go
package main

import (
    "fmt"
    "time"
)

func delay(n time.Duration) {
    duration := n * time.Second
    time.Sleep(duration)
    fmt.Println("Slept for ", duration)
}
```

Now in your main function, you can call the delay function directly and it will print the output after sleeping for `n` seconds. 

```go
func main() {
    delay(3)
}
```

But if you try to run the function as a separate goroutine, the output is empty.

```go
func main() {
    go delay(3)
}
```

The reason for that is once the main function invokes the delay function as a goroutine, there is no other statement to execute. 
So the main goroutine exits, before the delay function could print it's output. 

The dumb way to wait for the goroutine to finish, is to sleep in the main function after invoking the go routines. 

```go
func main() {
    for i := 1; i <= 3; i++ {
        go delay(time.Duration(i))
    }
    time.Sleep(4 * time.Second)
}
```

But this isn't the correct way to handle this. 
As we won't know in advance how many goroutines would be be run. 
And we need to wait for an extra 1 second after all goroutines are complete. 
And this solution feels icky. 


Go makes it easy to handle waiting for goroutines to complete by using a counting semaphone in the sync package called [`sync.WaitGroup`](https://golang.org/pkg/sync/#WaitGroup). 
Just before starting a goroutine, you need to `Add` to the waitgroup and once you are done, you call the `Done`. 
The `Add` increments the counter and the `Done` decrements the counter. 

Now you can use the waitgroup's `Wait` method to block the main function till the counter reaches 0. 
Let's see the same example, but rewritten with waitgroups. 

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func delay(n time.Duration, wg *sync.WaitGroup) {
    defer wg.Done()
    duration := n * time.Second
    time.Sleep(duration)
    fmt.Println("Slept for ", duration)
}

func main() {
    wg := new(sync.WaitGroup)
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go delay(time.Duration(i), wg)
    }
    wg.Wait()
}
```

We can see that our delay function is modified slightly to accept a `sync.WaitGroup` variable. And the last statement you want to be executed in the goroutine is the `wg.Done()` call. 

In our main function, we create a new WaitGroup variable and call the `wg.Add(1)` just before we spawn off the goroutine. And after invoking all the goroutines, we use the `wg.Wait()` to wait for all the goroutines to complete it's job. The main function now exits as soon as the last goroutine completes. 

We don't have to Add one at a time. If we know in advance how many goroutines are going to be run (say n int), we can use `wg.Add(n)` to increment the counter by n. 

There are other ways to controlling how and when a goroutine exits by using Channels, but WaitGroups are useful in specific places and it is better to learn this pattern.