---
title: "How to write benchmarks in Go"
date: 2018-12-12T23:13:00+05:30
lastmod: 2018-12-12T23:13:00+05:30
tags : ["benchmarks", "testing"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

In the previous post, we saw how to write a simple unit test case and also [how to write table driven test cases](/table-driven-unit-testing/). In this post we will quickly see how to write benchmarks, as go supports running benchmarks on your functions as part of the standard testing package. 

Using these simple benchmarks, you can quickly examine your code's performance. But do remember, that you would need a profiler to go in depth of which part of your code is causing the performance bottleneck.

<!--more-->

Let's use the Prime number function we wrote in the previous post for writing our benchmarks. 

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

## Writing your benchmark

Just like your test cases, benchmarks are placed inside the `_test.go` files, and instead of a function starting with `Test` it need to start with `Benchmark`.
Our benchmark function is very simple and very similar to your unit test cases. 

    func BenchmarkPrime(b *testing.B) {
        for i := 0; i < b.N; i++ {
            Prime(997)
        }
    }

The function needs to accept a `*testing.B` object and that has details about the number of iterations your benchmark needs to be run. 
The testing package runs the Benchmark functions several times. It keeps increasing the b.N value until the benchmark runner is satisfied with the stability of the benchmark. You must call your function b.N times, and that is why we are using a for loop to iterate till b.N times.

## Running benchmarks
The command to run benchmarks is the same as before, but with an extra flag.

    $ go test -bench=.
    goos: linux
    goarch: amd64
    pkg: github.com/cnu/prime
    BenchmarkPrime-4   	100000000	        18.3 ns/op
    PASS
    ok  	github.com/cnu/prime	1.847s

The output that we are interested in is the lines starting with Benchmark. This shows that the function `Prime(997)` was executed 100 Million times and on my desktop, it took 18.3 nanoseconds for each call. 

The PASS shows that the go test ran the test cases present in the package and it all passed. The last line shows how much time the entire benchmark command take to complete.

## Benchmarking different values

In our benchmark function, we were testing with only one value and testing for it. But what if we want to benchmark how it performs for various other numbers as input. We would need a benchmarking function which can take in arbitrary input. 

Our exported Benchmark function can only take in a single argument - a variable of type `*testing.B`. So we need to have a inner helper function which can take in the number we want to benchmark it with. 


    func benchmarkPrime(num uint, b *testing.B) {
        for i := 0; i < b.N; i++ {
            Prime(num)
        }
    }

This function is not exported, and takes in an extra argument `num` which is then passed to the Prime function call. Now we can make our BenchmarkPrime call to pass in this argument.


    func BenchmarkPrime1(b *testing.B)       { benchmarkPrime(1, b) }
    func BenchmarkPrime2(b *testing.B)       { benchmarkPrime(2, b) }
    func BenchmarkPrime3(b *testing.B)       { benchmarkPrime(3, b) }
    func BenchmarkPrime183(b *testing.B)     { benchmarkPrime(183, b) }
    func BenchmarkPrime923(b *testing.B)     { benchmarkPrime(923, b) }
    func BenchmarkPrime1039281(b *testing.B) { benchmarkPrime(1039281, b) }

It is best practice to name your Benchmark to be descriptive of the value you are testing with, so the output is easy to parse. The output is as shown below.

    $ go test -bench=.
    goos: linux
    goarch: amd64
    pkg: github.com/cnu/prime
    BenchmarkPrime1-4         	1000000000	         1.98 ns/op
    BenchmarkPrime2-4         	2000000000	         1.98 ns/op
    BenchmarkPrime3-4         	2000000000	         1.69 ns/op
    BenchmarkPrime183-4       	100000000	        18.2 ns/op
    BenchmarkPrime923-4       	20000000	        82.4 ns/op
    BenchmarkPrime1039281-4   	100000000	        18.1 ns/op
    PASS
    ok  	github.com/cnu/prime	15.307s

We can see that the time taken for each operation varies based on the number we pass. This gives us a much clearer picture of how performant our function. 

## Things to Note

When the benchmark is started, it is run for a value of b.N=1. If the function returns back within a second, it is re-run for 2, 5, 10, 20, 50, and so on. This will keep increasing till we reach a total of 1 second for the function to return. You do have the option to set a different benchmark time by using the `-benchtime` flag.

Since the benchmarking tool progressively increases the value of b.N, you should make sure that you don't use that number directly in your function calls. 

For eg:

    func BenchmarkPrimeWrong(b *testing.B) {
        for i := 0; i < b.N; i++ {
            Prime(uint(i))
        }
    }

In this example, the function wouldn't complete, because we are passing the value of i to the Prime function. And since this keeps increasing progressive, it never reaches a stable state.

    func BenchmarkPrimeWrong2(b *testing.B) {
            Prime(uint(b.N))
    }

And in this second example, we are passing the value b.N itself to the Prime function. This just runs it once and doesn't really runs any benchmark. The output would be 0ns/op.

## Conclusion

Go provides nice tools to test and benchmark your code that it makes your code self-documenting. And any change you make to the codebase, gets immediately tested, for both correctness and performance to catch any regressions. 