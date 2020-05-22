---
title: "Regular Expressions in Go"
date: 2018-12-10T23:13:00+05:30
lastmod: 2018-12-10T23:13:00+05:30
tags : ["regexp", "text processing"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---


Regular Expressions is one skill that is essential to any programmer. We deal with lot of unstructured data and regexps are one vital tool to parse and process it. 

![Regular Expressions - XKCD 208](https://imgs.xkcd.com/comics/regular_expressions.png)

Lets look at the different functions and methods available to construct and use regular expressions in Go. For this tutorial I am going to assume you know the basics of Regex. 

<!--more-->

## Basic MatchString

Golang has the [regexp](https://golang.org/pkg/regexp/) package, which makes matching regular expressions on strings easy. To do a basic substring match, we can use the [regexp.MatchString](https://golang.org/pkg/regexp/#MatchString) function. 

```go
matched, err := regexp.MatchString(`fo.`, "a quick brown fox jumped over the lazy dog")
fmt.Println(matched) // true
fmt.Println(err)     // nil (regexp is valid)
```

This regex matches the substring "fox" in the string. The variable matched is a boolean value depending on whether a match was found or not. You can use all the usual regex patterns like `^` and `$` to match beginning/ending words.

## Compile

If you are using the same pattern for matching in multiple places in your code, it makes sense to compile the pattern into a Regexp object. Use the [regexp.Compile](https://golang.org/pkg/regexp/#Compile) function to compile your pattern. If the pattern isn't a well-formed regular expression, the err variable will show it. If you use the [regexp.MustCompile](https://golang.org/pkg/regexp/#MustCompile) function, the program panics on an error in the expression.

While compiling Regexps, it makes sense to use raw strings so you don't have to escape your double quotes. But the other special characters need to be escaped with a backslash.

```go
re := regexp.MustCompile(`ba.?`)
fmt.Printf("%q\n", re.FindString("foo bar baz"))    // "bar"
fmt.Printf("%q\n", re.FindString("spam eggs"))      // ""
```

### FindString

Once a Regexp object is created, you can use the [FindString](https://golang.org/pkg/regexp/#Regexp.FindString) method to match the patterns. It returns the first matching string if there is a successful match, else it returns an empty string.

If you want to find the starting and ending of the match, you can use the FindStringIndex method. It returns a slice of integers to show the matching positions. 

```go
re := regexp.MustCompile(`ab?`)
fmt.Println(re.FindStringIndex("table"))        // [1 3]
fmt.Println(re.FindStringIndex("foo") == nil)   // true
```

### FindAllString
And to find all the matches, you can use the [FindAllString](https://golang.org/pkg/regexp/#Regexp.FindAllString) method, which returns a slice of strings as the output. If there are no matches, you get back an empty slice. The method takes an integer argument n; if n >= 0, the function returns at most n matches. If you pass -1, it returns all matches.

```go
re := regexp.MustCompile(`fo.`)
fmt.Printf("%q\n", re.FindAllString("food fox foobar", -1)) // ["foo" "fox" "foo"]
fmt.Printf("%q\n", re.FindAllString("food fox foobar", 2))  // ["foo" "fox"]
fmt.Printf("%q\n", re.FindAllString("spam eggs", -1))       // []
```

### Replace

Now that you know how to find strings based on a pattern, you would like to know how to replace strings. You can use the ReplaceAllString method to replace the text of all matches. It returns a new string, with all the matches of the regexp replaced with a replacement string. You can use the $ symbols in the replacement string to show you need to expand the matched group. 

```go
re := regexp.MustCompile("a(x*)b")
fmt.Println(re.ReplaceAllString("-ab-axxb-", "T"))      // -T-T-
fmt.Println(re.ReplaceAllString("-ab-axxb-", "$1"))     // --xx-
```

In this example, we have a pattern, which also has a group `(x*)` which matches 0 or more x characters. In the first replacement example, we are replace both `ab` and `axxb` with the letter `T` - simple enough.
In the second example we use `$1` to replace with the matched first group (in this case xx). That is why we get `--xx-` as the output. 

### Split

Another common function that is used a lot when you use regular expressions is the [Split](https://golang.org/pkg/regexp/#Regexp.Split) function. It splits a string into multiple slices of strings based on the pattern it matches. Just like `FindAllString` method, this also takes an integer n; if n >= 0, the method returns at most n strings. If you pass -1, it returns all split strings.

```go
a := regexp.MustCompile(`a`)
fmt.Printf("%q\n", a.Split("banana", -1)) // ["b" "n" "n" ""]
fmt.Printf("%q\n", a.Split("banana", 0))  // [] (nil slice)
fmt.Printf("%q\n", a.Split("banana", 1))  // ["banana"]
fmt.Printf("%q\n", a.Split("banana", 2))  // ["b" "nana"]

zp := regexp.MustCompile(`z+`)
fmt.Printf("%q\n", zp.Split("pizza", -1)) // ["pi" "a"]
fmt.Printf("%q\n", zp.Split("pizza", 0))  // [] (nil slice)
fmt.Printf("%q\n", zp.Split("pizza", 1))  // ["pizza"]
fmt.Printf("%q\n", zp.Split("pizza", 2))  // ["pi" "a"]
```

These are the most commonly used functions and methods related to Regular expressions in Go. What other functions do you use? Can you give any more examples?

**Pro Tip:** Use the site [RegexPal](https://www.regexpal.com/) to construct and debug your pattern