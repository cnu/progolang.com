---
title: "Listing Files and Directories in Go"
date: 2018-12-06T22:35:00+05:30
lastmod: 2018-12-06T22:35:00+05:30
tags : ["files", "directories"]
categories : [ "howto" ]
layout: post
highlightjslanguages: ["go"]
---

Next to reading/writing to files, learning how to list the different files and directories is very important. I used to use python for doing lot of string processing and the common way to store data is in the form of jsonl files in a directory. And you can easily go through each file in the directory by using the `glob.glob` module. 

<!--more-->

## Glob function

Go also has a [Glob](https://golang.org/pkg/path/filepath/#Glob) function which does exactly the same. Here is the signature of the function. 

    func Glob(pattern string) (matches []string, err error)

It takes a string containing the glob pattern and returns back a slice of filenames. Here is a sample program which gets all files ending with `.jsonl` extension in `/data/` directory. 

    package main

    import (
        "fmt"
        "path/filepath"
    )

    func main() {
        matches, _ := filepath.Glob("/data/*.jsonl")
        for _, match := range matches {
            fmt.Println(match)
        }
    }


## Walk

Glob is useful if you already know the pattern of the file you want to process. But if you want to go through every file and directory, `filepath.Walk` is a better approach. 

Here is the signature of the [Walk](https://golang.org/pkg/path/filepath/#Walk) function.

    func Walk(root string, walkFn WalkFunc) error

You start with a root directory from which you want to start the walk and for each file/directory in the root directory, the walkFn gets called. Now WalkFunc is a type of function which takes in the path, fileinfo and error and does something with the file that gets passed in.

    type WalkFunc func(path string, info os.FileInfo, err error) error

This sounds complicated, but lets see a simple example to make things clear.


    package main

    import (
        "fmt"
        "os"
        "path/filepath"
    )

    func WalkAllFilesInDir(dir string) error {
        return filepath.Walk(dir, func(path string, info os.FileInfo, e error) error {
            if e != nil {
                return e
            }

            // check if it is a regular file (not dir)
            if info.Mode().IsRegular() {
                fmt.Println("file name:", info.Name())
                fmt.Println("file path:", path)
            }
            return nil
        })
    }

    func main() {
        WalkAllFilesInDir("/home/username/go/src/github.com/")
    }

If you look at the main function, it is pretty simple - you just pass a directory to the `WalkAllFilesInDir` function. 
But WalkAlFilesInDir just calls the Walk function, with the dir string and an anonymous function of type WalkFunc. 

As mentioned before, WalkFunc takes the path, info and an error parameter. 
First we check if there is any error. Then we use the info.Mode().IsRegular() to find if its a regular file or a directory. 
If it's a file, we just print the file name and the file path. Else, we skip to the next file. 

The output looks something like this 

    file path: /home/username/go/src/github.com/uudashr/gopkgs/Makefile
    file name: README.md
    file path: /home/username/go/src/github.com/uudashr/gopkgs/README.md
    file name: main.go
    file path: /home/username/go/src/github.com/uudashr/gopkgs/cmd/gopkgs/main.go
    file name: doc.go

You can see that the Walk function goes through each and every subdirectory. This is an easy way to literally walk through all of your files and folders and process it. 


These are the two popular functions I use to traverse files and directories. Do you have any other functions which does these better? Leave them in the comments below.