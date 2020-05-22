---
title: "Table Driven Unit Testing"
date: 2018-12-11T22:25:00+05:30
lastmod: 2018-12-11T22:25:00+05:30
tags : ["testing"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

Go has built-in support for writing test cases for your code. And it is important to write as much test cases as possible to make sure you cover all possible conditions. You have to use the testing standard package to write your unit test cases. 

Let's first see the simplest way to write test cases for your functions, then see how to test multiple conditions. 

<!--more-->

For this example, we will use a simple function which tests whether a given number is prime or not. The code itself isn't optimized, but we will ignore that for now.

```go
package prime

func Prime(number uint) bool {
    switch number {
    case 0:
        return false
    case 1:
        return false
    }
    var i uint = 2
    for i < number {
        if number%i == 0 {
            return false
        }
        i++
    }
    return true
}
```

## Simple Test Case

Go expects your test cases to be in a file which ends in `_test.go`. We will create a file named prime_test.go in the same package as our original function. In this we can have our test cases in a function which starts with `Test`, which allows the go test command to run these functions as test cases.

Here is a simple function to test our Prime number checker.

```go
func TestPrime(t *testing.T) {
    res := Prime(2)
    if res != true {
        t.Errorf("Prime(2) = %t, expected true", res)
    }
}
```

We are checking if 2 is a prime or not. We are testing the result from the Prime function and returning an error, if the result doesn't match the expected value.

If we want to test a decent set of test cases for numbers like 0, 1, 4, 5, 6, etc., we have to write all those as separate function calls and check for the result individually. 

Or we can use Table Driven Unit Testing. 

## Table Driven Unit Test Cases

If we have a list of possible test cases to test, we can create a slice of those test cases, with the expected output to avoid rewriting most of the boilerplate testing code. 

First, lets create a slice of structs that we can use to store our test cases and the expected result.

```go
var tests = []struct {
    n   uint
    exp bool
}{
    {0, false},
    {1, false},
    {2, true},
    {3, true},
    {4, false},
    {5, true},
    {6, false},
    {49, false},
    {97, true},
    {997, true},
}
```

Since this Prime function is very simple, takes in one argument and returns a boolean, our struct matches the same data types. Now we have a list of test cases that we can iterate through and test one by one. 


To do that, let's replace the TestPrime function with this new TestPrime function.

```go
func TestPrime(t *testing.T) {
    for _, e := range tests {
        res := Prime(e.n)
        if res != e.exp {
            t.Errorf("Prime(%d) = %t, expected %t",
                e.n, res, e.exp)
        }
    }
}
```

We are iterating through the tests slice, calling Prime with the number and storing the result back in a variable. Then we check the obtained result with the expected result from the struct. If they don't match, we use the `Errorf` function of the `t` object to print a nice error message. 

Since the checking code and error printing code is the same, we are avoiding repeating a lot of code. 
Now whenever we need to add more test cases, you just append to the tests slice. This is a common pattern of writing unit test cases you would see in most Go code base. 
