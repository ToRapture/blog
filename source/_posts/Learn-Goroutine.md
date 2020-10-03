---
title: Learn Goroutine
date: 2020-04-17 15:48:00
mathjax: true
tags:
- Golang
---

## Definitions
https://morsmachine.dk/go-scheduler

An OS thread, which is the thread of execution managed by the OS and works pretty much like your standard POSIX thread. In the runtime code, it's called M for machine.

A goroutine, which includes the stack, the instruction pointer and other information important for scheduling goroutines, like any channel it might be blocked on. In the runtime code, it's called a G.

A context for scheduling, which you can look at as a localized version of the scheduler which runs Go code on a single thread. It's the important part that lets us go from a N:1 scheduler to a M:N scheduler. In the runtime code, it's called P for processor.

## Codes
```go
package main

import (
	"fmt"
	"math/rand"
	"net/http"
	"os"
	"sync"
	"time"

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

func calculate() {
	x := rand.Int()
	for i := 0; i < 1000000; i++ {
		x ^= rand.Int()
		x |= rand.Int()
		x &= rand.Int()
	}
}

func cpuIntenseTask() {
	logger.Infof("[cpuIntenseTask] started")

	wg := sync.WaitGroup{}
	for i := 0; i < 16; i++ {
		go func() {
			wg.Add(1)
			calculate()
			wg.Done()
		}()
	}
	wg.Wait()

	logger.Infof("[cpuIntenseTask] finished")
}

func sleepTask() {
	logger.Infof("[sleepTask] started")

	wg := sync.WaitGroup{}
	for i := 0; i < 64; i++ {
		go func() {
			wg.Add(1)
			time.Sleep(1500 * time.Millisecond)
			wg.Done()
		}()
	}
	wg.Wait()

	logger.Infof("[sleepTask] finished")
}

func httpGet() {
	resp, err := http.Get("https://google.com")
	if err != nil {
		fmt.Printf("httpGet error: %v\n", err)
		return
	}
	logger.Infof("httpGet status: %v", resp.StatusCode)
}

func netTask() {
	logger.Infof("[netTask] started")

	wg := sync.WaitGroup{}
	for i := 0; i < 31200; i++ {
		go func() {
			wg.Add(1)
			httpGet()
			wg.Done()
		}()
	}
	wg.Wait()

	logger.Infof("[netTask] finished")
}

func main() {
	cpuIntenseTask()
	sleepTask()
	netTask()
}
```
```
$ go build
$ GODEBUG=schedtrace=1000 ./go-learn 1>learn.out 2>&1
$ cat learn.out | grep -v "error"
SCHED 0ms: gomaxprocs=4 idleprocs=2 threads=4 spinningthreads=1 idlethreads=0 runqueue=0 [1 0 0 0]
[2020-04-17 15:40:53]  INFO cpuIntenseTask: started
[2020-04-17 15:40:53]  INFO cpuIntenseTask: finished
[2020-04-17 15:40:53]  INFO sleepTask: started
[2020-04-17 15:40:53]  INFO sleepTask: finished
[2020-04-17 15:40:53]  INFO netTask: started
SCHED 100ms: gomaxprocs=4 idleprocs=0 threads=10 spinningthreads=0 idlethreads=1 runqueue=19587 [128 25 0 0]
SCHED 204ms: gomaxprocs=4 idleprocs=0 threads=10 spinningthreads=0 idlethreads=1 runqueue=23291 [30 74 33 1]
SCHED 313ms: gomaxprocs=4 idleprocs=0 threads=10 spinningthreads=0 idlethreads=1 runqueue=14978 [77 41 108 125]
SCHED 421ms: gomaxprocs=4 idleprocs=0 threads=11 spinningthreads=0 idlethreads=0 runqueue=6388 [97 74 34 109]
SCHED 529ms: gomaxprocs=4 idleprocs=0 threads=12 spinningthreads=0 idlethreads=1 runqueue=0 [3 6 2 1]
SCHED 638ms: gomaxprocs=4 idleprocs=0 threads=12 spinningthreads=0 idlethreads=1 runqueue=4 [3 0 1 9]
SCHED 743ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=3 runqueue=0 [1 0 1 4]
SCHED 853ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=0 [2 7 1 2]
SCHED 960ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=160 [0 1 39 11]
SCHED 1070ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=129 [0 1 3 6]
SCHED 1179ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=93 [20 3 24 14]
SCHED 1286ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=1549 [0 1 74 114]
SCHED 1396ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=1597 [0 0 26 88]
SCHED 1503ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=426 [96 1 44 69]
 112 4]
SCHED 1716ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=41 [9 2 69 6]
SCHED 1821ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=62 [12 0 5 1]
SCHED 1929ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=51 [9 0 6 1]
SCHED 2036ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=45 [15 0 9 1]
SCHED 2139ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=88 [1 10 7 6]
SCHED 2243ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=135 [1 1 13 11]
SCHED 2349ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=51 [1 10 4 5]
SCHED 2456ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=44 [1 7 11 0]
SCHED 2563ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=63 [1 6 10 0]
SCHED 2670ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=3 runqueue=95 [1 6 10 0]
SCHED 2770ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=83 [1 7 9 0]
SCHED 2877ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=1 [1 12 155 1]
SCHED 2981ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=105 [11 11 13 1]
SCHED 3087ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=140 [7 11 1 1]
SCHED 3195ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=73 [1 0 8 0]
SCHED 3303ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=138 [46 0 4 1]
SCHED 3413ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=147 [22 0 0 1]
SCHED 3514ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=3 runqueue=157 [5 19 51 1]
1957 [5 86 124 1]
SCHED 3728ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=133 [3 38 38 0]
SCHED 3834ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=141 [40 31 14 0]
SCHED 3937ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=37 [1 2 8 2]
SCHED 4041ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=74 [0 1 9 0]
SCHED 4147ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=60 [0 1 14 9]
SCHED 4251ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=71 [0 0 9 41]
SCHED 4357ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=5 [0 1 0 0]
SCHED 4460ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=4 [0 2 0 1]
SCHED 4567ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=202 [21 1 0 11]
SCHED 4677ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=6 runqueue=79 [7 1 8 5]
4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=46 [4 1 1 2]
SCHED 4887ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=8 [0 0 0 0]
SCHED 4996ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=18 [0 0 1 3]
SCHED 5102ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=76 [15 0 1 11]
SCHED 5209ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=109 [70 1 1 1]
SCHED 5314ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=87 [12 1 1 4]
SCHED 5419ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=14 [0 0 1 1]
SCHED 5522ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=8 [0 0 1 1]
SCHED 5622ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=3 runqueue=90 [140 1 22 0]
SCHED 5725ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=85 [91 1 9 21]
SCHED 5830ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=71 [1 1 21 2]
SCHED 5936ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=8 [1 1 0 0]
SCHED 6038ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=99 [11 2 1 0]
SCHED 6143ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=14 [97 1 4 2]
SCHED 6249ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=55 [1 62 123 33]
SCHED 6355ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=78 [1 6 4 17]
SCHED 6458ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=485 [1 100 101 111]
SCHED 6563ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=101 [1 15 5 13]
SCHED 6667ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=40 [1 2 74 0]
SCHED 6775ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=85 [1 6 8 98]
SCHED 6879ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=71 [1 14 1 1]
 8 108 0]
SCHED 7088ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=613 [14 102 43 100]
SCHED 7196ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=155 [1 101 7 13]
SCHED 7297ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=95 [1 18 7 100]
SCHED 7402ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=27 [1 2 2 4]
SCHED 7507ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=12 [119 0 0 1]
SCHED 7612ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=44 [2 8 14 1]
SCHED 7720ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=14 [4 1 2 2]
SCHED 7825ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=0 [164 0 1 0]
SCHED 7931ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=3 runqueue=36 [4 0 0 1]
SCHED 8041ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=42 [5 2 2 1]
SCHED 8143ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=62 [0 2 7 1]
SCHED 8243ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=2 runqueue=132 [1 239 26 23]
SCHED 8346ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=68 [2 4 3 4]
SCHED 8451ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=39 [6 1 1 0]
SCHED 8560ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=151 [146 0 1 9]
SCHED 8665ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=100 [1 8 0 25]
SCHED 8771ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=38 [2 1 1 1]
SCHED 8878ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=0 [1 0 0 115]
SCHED 8987ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=89 [6 17 5 3]
SCHED 9088ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=5 runqueue=901 [99 195 80 116]
SCHED 9193ms: gomaxprocs=4 idleprocs=0 threads=15 spinningthreads=0 idlethreads=4 runqueue=0 [0 0 0 0]
SCHED 9301ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 9404ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 9508ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 9615ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 9717ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 9817ms: gomaxprocs=4 idleprocs=2 threads=15 spinningthreads=1 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 9926ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 10027ms: gomaxprocs=4 idleprocs=2 threads=15 spinningthreads=1 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 10139ms: gomaxprocs=4 idleprocs=2 threads=15 spinningthreads=1 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 10241ms: gomaxprocs=4 idleprocs=3 threads=15 spinningthreads=0 idlethreads=8 runqueue=0 [0 0 0 0]
SCHED 10341ms: gomaxprocs=4 idleprocs=2 threads=15 spinningthreads=1 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 10443ms: gomaxprocs=4 idleprocs=3 threads=15 spinningthreads=1 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 10547ms: gomaxprocs=4 idleprocs=2 threads=15 spinningthreads=1 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 10653ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 10753ms: gomaxprocs=4 idleprocs=2 threads=15 spinningthreads=1 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 10865ms: gomaxprocs=4 idleprocs=3 threads=15 spinningthreads=0 idlethreads=8 runqueue=0 [0 0 0 0]
SCHED 10975ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=8 runqueue=0 [0 0 0 0]
SCHED 11087ms: gomaxprocs=4 idleprocs=3 threads=15 spinningthreads=0 idlethreads=8 runqueue=0 [0 0 0 0]
SCHED 11189ms: gomaxprocs=4 idleprocs=2 threads=15 spinningthreads=1 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 11291ms: gomaxprocs=4 idleprocs=2 threads=15 spinningthreads=1 idlethreads=7 runqueue=0 [0 0 0 0]
SCHED 11391ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 11501ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 11604ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 11713ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 11821ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 11930ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12041ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12141ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12247ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12354ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12456ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12558ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12666ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12769ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12878ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 12983ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 13083ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 13186ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 13290ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 13395ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 13504ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 13605ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 13706ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
SCHED 13808ms: gomaxprocs=4 idleprocs=4 threads=15 spinningthreads=0 idlethreads=9 runqueue=0 [0 0 0 0]
[2020-04-17 15:41:07]  INFO netTask: finished
```

- - -
```go
package main

import (
	"fmt"
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

func main() {
	ch := make(chan string)

	go func() {
		var s string
		fmt.Scanln(&s)
		ch <- s
	}()

	s := <-ch
	logger.Infof("input is: %v", s)
}
```
```
$ GODEBUG=schedtrace=500,scheddetail=1 ./go-learn 1>learn.out 2>&1
$ cat learn.out
SCHED 0ms: gomaxprocs=4 idleprocs=2 threads=3 spinningthreads=1 idlethreads=0 runqueue=0 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
  P0: status=1 schedtick=0 syscalltick=0 m=0 runqsize=0 gfreecnt=0
  P1: status=1 schedtick=0 syscalltick=0 m=2 runqsize=0 gfreecnt=0
  P2: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0
  P3: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0
  M2: p=1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
  M1: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
  M0: p=0 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=1
  G1: status=1() m=-1 lockedm=0
  G2: status=1() m=-1 lockedm=-1
SCHED 500ms: gomaxprocs=4 idleprocs=4 threads=5 spinningthreads=0 idlethreads=3 runqueue=0 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
  P0: status=0 schedtick=2 syscalltick=0 m=-1 runqsize=0 gfreecnt=0
  P1: status=0 schedtick=3 syscalltick=0 m=-1 runqsize=0 gfreecnt=0
  P2: status=0 schedtick=2 syscalltick=6 m=-1 runqsize=0 gfreecnt=0
  P3: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0
  M4: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=true lockedg=-1
  M3: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=true lockedg=-1
  M2: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=true lockedg=-1
  M1: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
  M0: p=-1 curg=34 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=-1
  G1: status=4(chan receive) m=-1 lockedm=-1
  G2: status=4(force gc (idle)) m=-1 lockedm=-1
  G17: status=4(GC sweep wait) m=-1 lockedm=-1
  G18: status=4(GC scavenge wait) m=-1 lockedm=-1
  G33: status=4(finalizer wait) m=-1 lockedm=-1
  G34: status=3() m=0 lockedm=-1
SCHED 1010ms: gomaxprocs=4 idleprocs=4 threads=5 spinningthreads=0 idlethreads=3 runqueue=0 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
  P0: status=0 schedtick=2 syscalltick=0 m=-1 runqsize=0 gfreecnt=0
  P1: status=0 schedtick=3 syscalltick=0 m=-1 runqsize=0 gfreecnt=0
  P2: status=0 schedtick=2 syscalltick=6 m=-1 runqsize=0 gfreecnt=0
  P3: status=0 schedtick=0 syscalltick=0 m=-1 runqsize=0 gfreecnt=0
  M4: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=true lockedg=-1
  M3: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=true lockedg=-1
  M2: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=true lockedg=-1
  M1: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
  M0: p=-1 curg=34 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=-1
  G1: status=4(chan receive) m=-1 lockedm=-1
  G2: status=4(force gc (idle)) m=-1 lockedm=-1
  G17: status=4(GC sweep wait) m=-1 lockedm=-1
  G18: status=4(GC scavenge wait) m=-1 lockedm=-1
  G33: status=4(finalizer wait) m=-1 lockedm=-1
  G34: status=3() m=0 lockedm=-1
[2020-04-17 15:55:36]  INFO input is: hello
```