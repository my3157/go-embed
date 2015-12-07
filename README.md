## Go Embed
Generates go code to embed resource files into your library or executable.
This is more suitable for web servers as it gzip compresses all the files
automatically and computes the hash so that it can be used for caching the
assets in the frontends.

```bash
go-embed v0.1.0
Generates go code to embed resource files into your library or executable

  Usage:
    -input  string  The path to the folder containing the assets
    -output string  The output filename
    -tag    string  The tag to use for the generated package

  example:
    go-embed -input public/ -output assets/main.go
```

You can use this to embed your css, js and images into a single executable.

This is similar to [go-bindata](https://github.com/jteeuwen/go-bindata).

This is similar to [pony-embed](https://github.com/pyros2097/pony-embed).

This is similar to [rust-embed](https://github.com/pyros2097/rust-embed).

## Installation
```
go get github.com/pyros2097/go-embed
```
## Requirements
* An `index.html` file in your input folder

## Documentation
You can directly access your files as constants from the assets package or
you can use this func to serve all files stored in your assets folder which is useful for webservers and has gzip compression and caching inbuilt. Just see the example as to how same caching and compression works in
production and development.
```go
assets.IndexHtml // direct access
assets.Asset(path, debug) (data, hash, contentType) // where debug is bool
```
The Asset func does not return an error like the rest of the resource embedding tools and thats because in normal SPA applications if a route is not found then you automatically redirect it to the root path.
Here in go-embed if a file or path is not found then we directly send the 
data for "index.html" which MUST be present in your input folder.
And the root path will always return the data for "index.html"

## Examples
A simple http server which serves its resources directly.

To see it in action,
`go run examples/main.go`

```go
package main

import (
  "net/http"

  "github.com/pyros2097/go-embed/examples/assets"
)

func main() {
  http.HandleFunc("/", func(res http.ResponseWriter, req *http.Request) {
    println("GET " + req.URL.String())
    data, hash, contentType := assets.Asset(req.URL.String(), false)
    res.Header().Set("Content-Encoding", "gzip")
    res.Header().Set("Content-Type", contentType)
    res.Header().Add("Cache-Control", "public, max-age=31536000")
    res.Header().Add("ETag", hash)
    if req.Header.Get("If-None-Match") == hash {
      res.WriteHeader(http.StatusNotModified)
    } else {
      res.WriteHeader(http.StatusOK)
      _, err := res.Write(data)
      if err != nil {
        panic(err)
      }
    }
  })
  println("Server running on 127.0.0.1:3000")
  http.ListenAndServe(":3000", nil)
}
```

Go Gophers!

The power is yours!