+++
date = "2018-12-02T14:04:00+05:30"
layout = "post"
title = "How to download files in Go"
highlightjslanguages = ["go"]
toc = ""
+++


The best way to learn a new programming language is to write small utility programs. Let's write one such program to download files via http in Go. 

I am going to show the entire source code first and then walk through each section of it. 

<!--more-->
## Download a file over http

    package main

    import (
        "fmt"
        "io"
        "net/http"
        "os"
    )

    func main() {
        if len(os.Args) != 3 {
            fmt.Println("usage: download url filename")
            os.Exit(1)
        }
        url := os.Args[1]
        filename := os.Args[2]

        err := DownloadFile(url, filename)
        if err != nil {
            panic(err)
        }

    }

    // DownloadFile will download a url and store it in local filepath.
    // It writes to the destination file as it downloads it, without
    // loading the entire file into memory.
    func DownloadFile(url string, filepath string) error {
        // Create the file
        out, err := os.Create(filepath)
        if err != nil {
            return err
        }
        defer out.Close()

        // Get the data
        resp, err := http.Get(url)
        if err != nil {
            return err
        }
        defer resp.Body.Close()

        // Write the body to file
        _, err = io.Copy(out, resp.Body)
        if err != nil {
            return err
        }

        return nil
    }

### main function

Since this is a command line tool, we start with the package main and function main. We ask the user to pass the url to download and the filename to save it under as command line arguments. We can access them using the os.Args slice.

We just pass the url and filename to the `DownloadFile` function. The only return value from this function is an error, which we display to the user and exit. 

### DownloadFile function

Now if we look at the `DownloadFile` function, this is pretty simple. 

1. We create a new file at the filepath location
2. Then we use `http.Get()` to do a GET request to the url.
3. Finally we use `io.Copy()` to read from the `resp.Body` into the newly created file `out`.
4. We make sure to close both the file and the response. 

Now how can we say that it is memory efficient and doesn't download and store the entire file in memory? For that we can just look at the source code of `io.Copy()` and follow along till we reach `copyBuffer()`. Here we can see that it create a buffer []byte array of [size 32kb](https://golang.org/src/io/io.go?s=13825:13845#L391) and uses that to read from the source Reader to destination Writer. 

## Better DownloadeFile

This does work well, but for this tool to be a bit more useful, it needs to show some output to the user. Some indication that it just downloaded a file successfully. Especially if the file is a few hundreds of MB in size, we don't want the command to keep waiting. 

Let's also add in an extra feature of first downloading to a temporary file so that we don't overwrite an old file till the new file is completely downloaded. 

### WriteCounter to count bytes

First off we are going to create a struct which can be used to count the number of bytes which is written. It has only a simple field of uint64. But what is interesting are the two methods that we are going to write for this struct. 


    type WriteCounter struct {
        Total uint64
    }


### Write() and PrintProgress()

We are implementing the Write method for this struct, which makes this object of the io.Writer interface. We can pass this object to any function which requires a io.Writer interface. And the Write function just increments the counter by the size of the bytes written into it and then calls the PrintProgress method. 

    func (wc *WriteCounter) Write(p []byte) (int, error) {
        n := len(p)
        wc.Total += uint64(n)
        wc.PrintProgress()
        return n, nil
    }

The PrintProgress method is just an utility method which prints how many bytes were written. It uses the [go-humanize](https://github.com/dustin/go-humanize) package to convert the number of bytes into a human readable number. 

    // PrintProgress prints the progress of a file write
    func (wc WriteCounter) PrintProgress() {
        // Clear the line by using a character return to go back to the start and remove
        // the remaining characters by filling it with spaces
        fmt.Printf("\r%s", strings.Repeat(" ", 50))

        // Return again and print current status of download
        // We use the humanize package to print the bytes in a meaningful way (e.g. 10 MB)
        fmt.Printf("\rDownloading... %s complete", humanize.Bytes(wc.Total))
    }

### DownloadFile function

Now there is just one minor change in the `DownloadFile` function. Instead of just passing `resp.Body` into `io.Copy`, we are creating an `io.TeeReader` which takes in the counter struct we created earlier. 
The `io.TeeReader`, requires one Reader and one Writer, and it returns a Reader by itself. It reads from `resp.Body`, writes to the `counter`, and then passes those bytes to the outer `io.Copy` function. This is a common pattern followed in Unix pipes. 

Other than the TeeReader, we are creating a temporary file to download our file, so that we don't overwrite an existing file before the entire file is downloaded. Once the download is complete, we rename the file names and complete our program. 

    func DownloadFile(url string, filepath string) error {

        // Create the file with .tmp extension, so that we won't overwrite a
        // file until it's downloaded fully
        out, err := os.Create(filepath + ".tmp")
        if err != nil {
            return err
        }
        defer out.Close()

        // Get the data
        resp, err := http.Get(url)
        if err != nil {
            return err
        }
        defer resp.Body.Close()

        // Create our bytes counter and pass it to be used alongside our writer
        counter := &WriteCounter{}
        _, err = io.Copy(out, io.TeeReader(resp.Body, counter))
        if err != nil {
            return err
        }

        // The progress use the same line so print a new line once it's finished downloading
        fmt.Println()

        // Rename the tmp file back to the original file
        err = os.Rename(filepath+".tmp", filepath)
        if err != nil {
            return err
        }

        return nil
    }

If you run the entire code to download a large file, this is how it shows up on the terminal.

![download tool demo](/uploads/download-demo.gif)

The source code of both the programs are available on this [gist link](https://gist.github.com/cnu/026744b1e86c6d9e22313d06cba4c2e9).

## Conclusion

This shows, how age-old patterns like the unix pipes and tee commands are still relevant and we can build simple tools using those patterns. But more important is the way interfaces are implemented in golang, which allows you to create new structures like the `WriteCounter` and pass them to any place which takes in an `io.Writer` interface. 