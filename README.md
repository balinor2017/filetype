# filetype [![Build Status](https://travis-ci.org/haxii/filetype.svg?branch=master)](https://travis-ci.org/haxii/filetype)[![GoDoc](https://godoc.org/github.com/haxii/filetype?status.svg)](https://godoc.org/github.com/haxii/filetype)
# filetype  [![Build Status](https://travis-ci.org/haxii/filetype.svg?branch=master)](https://travis-ci.org/haxii/filetype) [![GoDoc](https://godoc.org/github.com/haxii/filetype?status.svg)](https://godoc.org/github.com/haxii/filetype)

Small and dependency free [Go](https://golang.org) package to infer file and MIME type checking the [magic numbers](https://en.wikipedia.org/wiki/Magic_number_(programming)#Magic_numbers_in_files) signature.

## Features

- Supports a [wide range](#supported-types) of file types
- Provides file extension and proper MIME type
- File discovery by extension or MIME type
- File discovery by class (image, video, audio...)
- Provides a bunch of helpers and file matching shortcuts
- [Pluggable](#add-additional-file-type-matchers): add custom new types and matchers
- Simple and semantic API
- [Blazing fast](#benchmarks), even processing large files
- Only first 261 bytes representing the max file header is required, so you can just [pass a slice](#file-header)
- Dependency free (just Go code, no C compilation needed)
- Cross-platform file recognition

## Installation

```bash
go get github.com/haxii/filetype
```

## API

See [Godoc](https://godoc.org/github.com/haxii/filetype) reference.

### Subpackages

- [`github.com/haxii/filetype/types`](https://godoc.org/github.com/haxii/filetype/types)
- [`github.com/haxii/filetype/matchers`](https://godoc.org/github.com/haxii/filetype/matchers)

## Examples

### Simple file type checking

```go
package main

import (
  "fmt"
  "github.com/haxii/filetype"
  "io/ioutil"
)

func main() {
  buf, _ := ioutil.ReadFile("sample.jpg")

  kind, unknown := filetype.Match(buf)
  if unknown != nil {
    fmt.Printf("Unknown: %s", unknown)
    return
  }

  fmt.Printf("File type: %s. MIME: %s\n", kind.Extension, kind.MIME.Value)
}
```

#### Check type class

```go
package main

import (
  "fmt"
  "github.com/haxii/filetype"
  "io/ioutil"
)

func main() {
  buf, _ := ioutil.ReadFile("sample.jpg")

  if filetype.IsImage(buf) {
    fmt.Println("File is an image")
  } else {
    fmt.Println("Not an image")
  }
}
```

#### Supported type

```go
package main

import (
  "fmt"
  "github.com/haxii/filetype"
)

func main() {
  // Check if file is supported by extension
  if filetype.IsSupported("jpg") {
    fmt.Println("Extension supported")
  } else {
    fmt.Println("Extension not supported")
  }

  // Check if file is supported by extension
  if filetype.IsMIMESupported("image/jpeg") {
    fmt.Println("MIME type supported")
  } else {
    fmt.Println("MIME type not supported")
  }
}
```

#### File header

```go
package main

import (
  "fmt"
  "github.com/haxii/filetype"
  "io/ioutil"
)

func main() {
  // Open a file descriptor
  file, _ := os.Open("movie.mp4")

  // We only have to pass the file header = first 261 bytes
  head := make([]byte, 261)
  file.Read(head)

  if filetype.IsImage(head) {
    fmt.Println("File is an image")
  } else {
    fmt.Println("Not an image")
  }
}
```

#### Add additional file type matchers

```go
package main

import (
  "fmt"
  "github.com/haxii/filetype"
)

var fooType = filetype.NewType("foo", "foo/foo")

func fooMatcher(buf []byte) bool {
  return len(buf) > 1 && buf[0] == 0x01 && buf[1] == 0x02
}

func main() {
  // Register the new matcher and its type
  filetype.AddMatcher(fooType, fooMatcher)

  // Check if the new type is supported by extension
  if filetype.IsSupported("foo") {
    fmt.Println("New supported type: foo")
  }

  // Check if the new type is supported by MIME
  if filetype.IsMIMESupported("foo/foo") {
    fmt.Println("New supported MIME type: foo/foo")
  }

  // Try to match the file
  fooFile := []byte{0x01, 0x02}
  kind, _ := filetype.Match(fooFile)
  if kind == filetype.Unknown {
    fmt.Println("Unknown file type")
  } else {
    fmt.Printf("File type matched: %s\n", kind.Extension)
  }
}
```

## Supported types

### Image

- **jpg** - `image/jpeg`
- **png** - `image/png`
- **gif** - `image/gif`
- **webp** - `image/webp`
- **cr2** - `image/x-canon-cr2`
- **tif** - `image/tiff`
- **bmp** - `image/bmp`
- **jxr** - `image/vnd.ms-photo`
- **psd** - `image/vnd.adobe.photoshop`
- **ico** - `image/x-icon`
- **jb2** - `image/x-jb2`
- **jfif** - `image/jpeg`
- **jpe** - `image/jpeg`
- **jpg** - `image/jpeg`
- **jp2** - `image/jp2`
- **vhd** - `application/octet-stream`
- **wmf** - `windows/metafile`
- **jb2** - `image/x-jb2`

#### Video

- **mp4** - `video/mp4`
- **m4v** - `video/x-m4v`
- **mkv** - `video/x-matroska`
- **webm** - `video/webm`
- **mov** - `video/quicktime`
- **avi** - `video/x-msvideo`
- **wmv** - `video/x-ms-wmv`
- **mpg** - `video/mpeg`
- **flv** - `video/x-flv`
- **mpg** - `video/x-ms-vob`
- **rmvb** - `application/vnd.rn-realmedia`
- **vcd** - `application/x-cdlink`
- **vmd** - `chemical/x-vmd`
- **wpl** - `application/vnd.ms-wpl`
- **dvr** - `video/x-ms-dvr`
- **ivr** - `video/vnd.rn-realvideo`
- **3gg** - `video/3gpp`
- **3gp** - `video/3gpp`
- **3g2** - `video/3gpp`
- **wmv** - `video/x-ms-asf`

#### Audio

- **mid** - `audio/midi`
- **mp3** - `audio/mpeg`
- **m4a** - `audio/m4a`
- **ogg** - `audio/ogg`
- **flac** - `audio/x-flac`
- **wav** - `audio/x-wav`
- **amr** - `audio/amr`
- **aac** - `audio/aac`
- **wma** - `audio/x-ms-wma`
- **caf** - `audio/x-caf`
- **cda** - `application/x-cdf`
- **qcp** - `audio/vnd.qcelp`
- **ra** - `audio/x-realaudio`
- **rm** - `audio/x-pn-realaudio`
- **rmi** - `audio/mid`
- **voc** - `audio/voc`

#### Archive

- **zip** - `application/zip`
- **tar** - `application/x-tar`
- **rar** - `application/x-rar-compressed`
- **gz** - `application/gzip`
- **bz2** - `application/x-bzip2`
- **7z** - `application/x-7z-compressed`
- **xz** - `application/x-xz`
- **pdf** - `application/pdf`
- **exe** - `application/x-msdownload`
- **swf** - `application/x-shockwave-flash`
- **rtf** - `application/rtf`
- **ps** - `application/postscript`
- **sqlite** - `application/x-sqlite3`
- **nes** - `application/x-nintendo-nes-rom`
- **crx** - `application/x-google-chrome-extension`
- **cab** - `application/vnd.ms-cab-compressed`
- **ar** - `application/x-unix-archive`
- **Z** - `application/x-compress`
- **lz** - `application/x-lzip`
- **rpm** - `application/x-rpm`

#### Documents

- **xls** - `application/vnd.ms-excel`
- **ppt** - `application/vnd.ms-powerpoint`
- **fdf** - `application/vnd.fdf`
- **ai** - `application/postscript`
- **tsa** - `text/tab-separated-values`
- **wim** - `application/octet-stream`

#### Font

- **woff** - `application/font-woff`
- **woff2** - `application/font-woff`
- **ttf** - `application/font-sfnt`
- **otf** - `application/font-sfnt`

## Benchmarks

Measured using [real files](https://github.com/haxii/filetype/tree/master/fixtures).

Environment: OSX x64 Xeon 2.8 Ghz

```bash
BenchmarkMatchTar-8    500000       2700 ns/op
BenchmarkMatchZip-8    500000       2933 ns/op
BenchmarkMatchJpeg-8   500000       2507 ns/op
BenchmarkMatchGif-8    500000       2923 ns/op
BenchmarkMatchPng-8    500000       2569 ns/op
BenchmarkMatchM4v-8    500000       2472 ns/op
BenchmarkMatchTif-8    500000       2742 ns/op
```

## License

MIT - Tomas Aparicio
