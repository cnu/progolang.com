---
title: "String Replacements in Go"
date: 2018-12-14T22:02:00+05:30
lastmod: 2018-12-14T22:02:00+05:30
tags : ["strings", "text processing"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

In a previous post, we saw how we can use [Regular expressions in Go](/regular-expressions-in-go/) to match and replace patterns in a string. While regexps are very useful, you might not want to them for every time you want to do some basic string manipulation. The `strings` package has few functions which works well for basic string replacement.

Let's look at two such functions.

<!--more-->

## strings.Replace

Just as the name suggests, this [`strings.Replace`](https://golang.org/pkg/strings/#Replace) function does a very basic string replacement. This is the signature of this function.

    func Replace(s, old, new string, n int) string

`s` is the string in which you want to replace `old` with `new` and the integer `n` tells how many occurences should be replaced. If `n` is < 0, all occurences will be replaced.

    package main

    import (
        "strings"
        "fmt"
    )

    const refString = "Apple is the most customer centric company. Apple is the first company to reach 1 Trillion market cap in the world."
    func main() {
        out := strings.Replace(refString, "Apple", "Amazon", 1)
        fmt.Println(out)
    }

As expected, this prints the following.

    Amazon is the most customer centric company. Apple is the first company to reach 1 Trillion market cap in the world.

## strings.NewReplacer

The `strings.Replace` works fine if you have only one string to replace. If you have multiple strings, you would need something more powerful. Thats why we have the [`strings.NewReplacer`](https://golang.org/pkg/strings/#NewReplacer). The NewReplacer function takes in pairs of old:new strings and returns a new Replacer. You can call the Replace method on this struct to do a quick replacement on your string.

    package main

    import (
        "fmt"
        "strings"
    )

    func main() {
        r := strings.NewReplacer("<", "&lt;", ">", "&gt;")
        fmt.Println(r.Replace("This is <b>HTML</b>!"))
    }

The advantage of using the strings.NewReplacer is that it uses an efficient data structure (called Trie) to do all the replacements in a single pass of the string. This is much faster if you have a lot of very basic string replacements. 

However if you need to use various patterns to match, you are stuct with constructing and compiling your regular expression. 