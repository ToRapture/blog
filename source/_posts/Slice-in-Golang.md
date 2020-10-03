---
title: Slice in Golang
date: 2020-03-27 16:24:00
tags: Golang
---

## Blogs
[Go Slices: usage and internals](https://blog.golang.org/slices-intro)
[Arrays, slices (and strings): The mechanics of 'append'](https://blog.golang.org/slices)

## Details
### make
Look up in `$GOROOT/src/runtime/slice.go:makeslice`.

### append
https://stackoverflow.com/a/33405824/13133551

### slicing
> The length is the number of elements referred to by the slice. The capacity is the number of elements in the underlying array (beginning at the element referred to by the slice pointer).

### copy
Look up in `$GOROOT/src/runtime/slice.go:slicecopy`.
> The copy function supports copying between slices of different lengths (it will copy only up to the smaller number of elements). In addition, copy can handle source and destination slices that share the same underlying array, handling overlapping slices correctly.

## Codes
```go
package main

import (
	"os"

	logger "github.com/sirupsen/logrus"
	"github.com/x-cray/logrus-prefixed-formatter"
)

func assertPanic(v bool) {
	if !v {
		panic("v is not true")
	}
}

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

func sliceCopy() {
	logger.Infof("in sliceCopy")
	s := make([]int, 0)
	for i := 0; i < 10; i++ {
		s = append(s, i)
	}

	c := make([]int, 0)
	copy(c, s)
	logger.Infof("c: %v", c)

	c = make([]int, 5)
	copy(c, s)
	logger.Infof("c: %v", c)

	c = make([]int, 15)
	copy(c, s)
	logger.Infof("c: %v", c)

	c = make([]int, 5, 20)
	copy(c, s)
	logger.Infof("c: %v", c)

	c[0] = 256
	logger.Infof("c: %v, s: %v", c, s)
}

func sliceAppend() {
	logger.Infof("in sliceAppend")
	s := make([]int, 0)
	for i := 0; i < 10; i++ {
		s = append(s, i)
		logger.Infof("after appending: %v, len(s): %v, cap(s): %v", i, len(s), cap(s))
	}
	for i := 0; i < 10; i++ {
		l := len(s)
		c := cap(s)
		after := append(s, i)
		if l == c {
			assertPanic(&s[0] != &after[0])
			assertPanic(s[0] == after[0])

			s[0] += 8
			assertPanic(s[0] != after[0])
		} else {
			assertPanic(&s[0] == &after[0])
			assertPanic(s[0] == after[0])

			s[0] += 8
			assertPanic(s[0] == after[0])
		}
		s = after
	}
}

func resultOfAppend() {
	logger.Infof("in resultOfAppend")
	s := make([]int, 0, 64)
	s = append(s, 1, 2, 3, 4, 5)

	x := append(s, 16)
	assertPanic(x[5] == 16)

	y := append(s, 32)
	assertPanic(x[5] == 32)
	assertPanic(y[5] == 32)
}

func slicing() {
	logger.Infof("in slicing")
	s := make([]int, 0, 6)
	s = append(s, 1, 2, 3, 4, 5)

	y := s[1:3]
	assertPanic(len(y) == 2)
	assertPanic(cap(y) == 5)

	y[0] = 10
	assertPanic(s[1] == 10)

	y = append(y, 8)
	assertPanic(s[3] == 8)
	assertPanic(y[2] == 8)

	s[3] = 32
	assertPanic(y[2] == 32)
}

func main() {
	sliceCopy()
	sliceAppend()
	resultOfAppend()
	slicing()
}
```
Result:
```
[2020-03-27 16:04:59]  INFO in sliceCopy
[2020-03-27 16:04:59]  INFO c: []
[2020-03-27 16:04:59]  INFO c: [0 1 2 3 4]
[2020-03-27 16:04:59]  INFO c: [0 1 2 3 4 5 6 7 8 9 0 0 0 0 0]
[2020-03-27 16:04:59]  INFO c: [0 1 2 3 4]
[2020-03-27 16:04:59]  INFO c: [256 1 2 3 4], s: [0 1 2 3 4 5 6 7 8 9]
[2020-03-27 16:04:59]  INFO in sliceAppend
[2020-03-27 16:04:59]  INFO after appending: 0, len(s): 1, cap(s): 1
[2020-03-27 16:04:59]  INFO after appending: 1, len(s): 2, cap(s): 2
[2020-03-27 16:04:59]  INFO after appending: 2, len(s): 3, cap(s): 4
[2020-03-27 16:04:59]  INFO after appending: 3, len(s): 4, cap(s): 4
[2020-03-27 16:04:59]  INFO after appending: 4, len(s): 5, cap(s): 8
[2020-03-27 16:04:59]  INFO after appending: 5, len(s): 6, cap(s): 8
[2020-03-27 16:04:59]  INFO after appending: 6, len(s): 7, cap(s): 8
[2020-03-27 16:04:59]  INFO after appending: 7, len(s): 8, cap(s): 8
[2020-03-27 16:04:59]  INFO after appending: 8, len(s): 9, cap(s): 16
[2020-03-27 16:04:59]  INFO after appending: 9, len(s): 10, cap(s): 16
[2020-03-27 16:04:59]  INFO in resultOfAppend
[2020-03-27 16:04:59]  INFO in slicing

```