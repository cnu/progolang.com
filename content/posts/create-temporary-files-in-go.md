+++
date = "2018-12-03T01:48:00+05:30"
layout = "post"
title = "How to create temporary files in go"
highlightjslanguages = ["go"]
toc = ""
+++

When programming in python, one of the common modules I used often is the tempfile module. This module contains functions which create temporary files and returns a file object. This is useful if you are downloading large files and want to cleanup automatically once you are done. 

Go has similar functions. Lets look at a sample code which create temporary folders and files and writes to it.

<!--more-->

Here is the example source code of the program. 

    package main

    import (
        "fmt"
        "io/ioutil"
        "os"
    )

    func main() {

        // Create a Temp File:  This will create a filename like /tmp/pre-23423234
        // If we use a pattern like "pre-*.ext", you can get a file like: /tmp/pre-23423234.ext
        tmpFile, err := ioutil.TempFile(os.TempDir(), "pre-")
        if err != nil {
            fmt.Println("Cannot create temporary file", err)
        }

        // cleaning up by removing the file
        defer os.Remove(tmpFile.Name())

        fmt.Println("Created a Temp File: " + tmpFile.Name())

        // Write to the file
        text := []byte("Writing some text into the temp file.")
        if _, err = tmpFile.Write(text); err != nil {
            fmt.Println("Failed to write to temporary file", err)
        }
        
        // Close the file
        if err := tmpFile.Close(); err != nil {
            fmt.Println(err)
        }
    }

The [io/ioutil](https://golang.org/pkg/io/ioutil/) package contains two function [`TempDir`](https://golang.org/pkg/io/ioutil/#TempDir) and [`TempFile`](https://golang.org/pkg/io/ioutil/#TempFile) to create a temporary directory and a temporary file. 

By default on a unix machine, the TempDir function returns `/tmp/` as the temporary directory. You can pass in additional parameters to create directories with prefixes. 

The TempFile function creates a unique and random number. You can control the prefix and suffix of the file with the `pattern` parameter. By passing `example-*.tmp`, you will create temporary files. This is useful if you don't want to delete the temporary files in the program, but instead want to have a batch delete job which cleans up the temporary files later. 

Once the file is created, you can write anything into the file as it is of type `os.File`. And since os.File implements the `io.ReadWriter` interface, you can write into the file using the `Write` method.

