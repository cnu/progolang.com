---
title: "Parsing Date and Time in Go"
date: 2018-12-04T23:55:31+05:30
lastmod: 2018-12-04T23:55:31+05:30
tags : ["date", "datetime", "time"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

Parsing a string containing date and time is one common task any programmer would encounter. It is also one of the most frustrating one, if you can't remember the specific format string correctly (is if %b or %B?).

Go does datetime parsing slightly differently. Instead of having cryptic format strings, go uses something called layouts. There is one special date that you need to remember whenever you need to parse dates in go. 

> Mon Jan 2 15:04:05 MST 2006

Why this specific date? 

<!--more-->

If you write it down in this format, it's obvious. 

> 01/02 03:04:05PM ‘06 -0700.

It is January 2nd, 3:04:05 PM of 2006, UTC-0700.

## time.Parse and Time.Format

Now whenever you need to parse a date string, you have to use this layout and pass to the `time.Parse` function.
And if you have a `time.Time` type, you can use the `Format` function to convert it to a string using the same layout structure.

```go
input := "2018-04-24"
layout := "2006-01-02"
t, _ := time.Parse(layout, input)
fmt.Println(t)                       // 2018-04-24 00:00:00 +0000 UTC
fmt.Println(t.Format("02-Jan-2006")) // 24-Apr-2018
```

## Date and Time Options Cheatsheet

You can use this table below as a quick cheatsheet for the different date and time options and the layout params you need to use.


Type	    | Options
------------|---------
Year	    | 06,   2006
Month	    | 01,   1,   Jan,   January
Day	        | 02,   2,   _2,   (width two, right justified)
Weekday	    | Mon,   Monday
Hours	    | 03,   3,   15
Minutes	    | 04,   4
Seconds	    | 05,   5
ms μs ns	| .000,   .000000,   .000000000
ms μs ns	| .999,   .999999,   .999999999   (trailing zeros removed)
am/pm	    | PM,   pm
Timezone	| MST
Offset	    | -0700,   -07,   -07:00,   Z0700,   Z07:00


## Common Date Time formats

Common Formats                  | Layout
--------------------------------|--------------------------
Date                            | January 2, 2006
                                | 01/02/06
                                | Jan-02-06
Time                            | 15:04:05
                                | 3:04:05 PM	 
Timestamp with microseconds     | Jan _2 15:04:05
                                | Jan _2 15:04:05.000000
ISO 8601                        | 2006-01-02T15:04:05-0700
                                | 2006-01-02
                                | 15:04:05
RFC 822 with numeric zone       | 02 Jan 06 15:04 MST
                                | 02 Jan 06 15:04 -0700
RFC 1123 with numeric zone      | Mon, 02 Jan 2006 15:04:05 MST
                                | Mon, 02 Jan 2006 15:04:05 -0700

