---
title: Map in Golang
date: 2020-03-30 15:09:00
tags: Golang
---

## Reference
[Go maps in action](https://blog.golang.org/maps)
[Comparison operators](https://golang.org/ref/spec#Comparison_operators)

## Declaration and initialization
> A Go map type looks like this:
> `map[KeyType]ValueType`
> where KeyType may be any type that is comparable (more on this later), and ValueType may be any type at all, including another map!

## Key types
As mentioned earlier, map keys may be of any type that is comparable. The language spec defines this precisely, but in short, comparable types are boolean, numeric, string, pointer, channel, and interface types, and structs or arrays that contain only those types. Notably absent from the list are slices, maps, and functions; these types cannot be compared using ==, and may not be used as map keys.

## Code
```go
package main

import (
	"os"

	logger "github.com/sirupsen/logrus"
	"github.com/x-cray/logrus-prefixed-formatter"
)

func init() {
	logger.SetFormatter(&prefixed.TextFormatter{
		TimestampFormat: "2006-01-02 15:04:05",
		FullTimestamp:   true,
		ForceFormatting: true,
		DisableColors:   true,
	})
	logger.SetOutput(os.Stdout)
	logger.SetLevel(logger.DebugLevel)
}

type Foo struct {
	Y interface{}
}

func main() {
	mp := map[Foo]int{
		Foo{
			Y: [...]int{1, 2, 3},
		}: 256,
	}
	logger.Infof("mp: %+v", mp)

	mp = map[Foo]int{
		Foo{
			Y: map[int]int{},
		}: 256,
	}
	logger.Infof("mp: %+v", mp)
}
```
Results:
```
panic: runtime error: hash of unhashable type map[int]int
[2020-03-30 15:06:32]  INFO mp: map[{Y:[1 2 3]}:256]

goroutine 1 [running]:
main.main()
	/Users/torapture/Codes/repos/go/src/go-learn/main.go:33 +0x1a2
```