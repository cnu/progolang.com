---
title: "Accepting Variable Number of Arguments in a Function"
date: 2018-12-08T22:15:00+05:30
lastmod: 2018-12-08T22:15:00+05:30
tags : ["functions"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

Functions in Go are capable of accepting multiple number of arguments, also called as variadic functions. One prominent example of such a function is `fmt.Println` as you can pass in any number of variables, it will print each of them separated by a space.

<!--more-->

The way to define a variadic function is to prepend the data type with three dots. That variable is now accessible as a slice and can be iterated.

Here is a simple example which explains it.

    func sum(numbers ...int) int {
        total := 0
        for _, n := range numbers {
            total += n
        }
        return total
    }



This is a simple sum function, which iterated through all numbers and returns back the total. 
Now to call this function, you just pass all the numbers as separate arguments. 

	total := sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

By declaring the argument as a `...int`, your numbers variable is now passed into the function as a slice of integers. But what if you already have a slice of numbers and want to pass it to this function?

    numbers := []int{1,2,3,4,5}
    total := sum(numbers...)

You just have to pass in the slice followed by three dots to unwrap the slice into individual parameters to the function. 