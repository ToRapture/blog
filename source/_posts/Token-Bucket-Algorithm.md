---
title: Token Bucket Algorithm
date: 2020-03-31 16:04:00
tags:
- Design
---

## Reference
[Token bucket](https://en.wikipedia.org/wiki/Token_bucket)

> Over the long run the output of conformant packets is limited by the token rate, ${\displaystyle r}$.

## Implementation
```go
package main

import (
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

type TokenBucket struct {
	mu              sync.Mutex
	startTime       time.Time
	availableTokens int64
	capacity        int64
	fillInterval    time.Duration
	lastTick        int64
}

func NewTokenBucket(qps, capacity int64) *TokenBucket {
	return &TokenBucket{
		startTime:       time.Now(),
		availableTokens: capacity,
		capacity:        capacity,
		lastTick:        0,
		fillInterval:    time.Duration(int64(time.Second) / qps),
	}
}

func (tb *TokenBucket) adjust() {
	if (tb.availableTokens) >= tb.capacity {
		return
	}
	now := time.Now()
	tick := int64(now.Sub(tb.startTime) / tb.fillInterval)
	tb.availableTokens += tick - tb.lastTick
	if tb.availableTokens > tb.capacity {
		tb.availableTokens = tb.capacity
	}
	tb.lastTick = tick
}

func (tb *TokenBucket) TakeAvailable(count int64) int64 {
	tb.mu.Lock()
	defer tb.mu.Unlock()

	tb.adjust()

	if tb.availableTokens <= 0 {
		return 0
	}

	if count >= tb.availableTokens {
		count = tb.availableTokens
	}

	tb.availableTokens -= count
	return count
}

func task(qps, num int64, timeNeed time.Duration) {
	logger.Infof("task qps: %v, num: %v, timeNeed: %.3fs", qps, num, timeNeed.Seconds())
	bucket := NewTokenBucket(qps, qps)
	startTime := time.Now()
	lastTime := startTime

	for i := int64(1); i <= num; {
		if bucket.TakeAvailable(1) == 1 {
			time.Sleep(timeNeed)
			i++
			if i%1000 == 0 {
				now := time.Now()
				logger.Infof("rate: %.3f", 1000/now.Sub(lastTime).Seconds())
				lastTime = now
			}
		}
	}

	logger.Infof("task over, used: %.3fs", time.Now().Sub(startTime).Seconds())
}

func main() {
	task(10000, 100000, 0)
	task(10000, 15000, 2*time.Millisecond)
	task(500, 10000, 0)
}
```
```
[2020-03-31 15:58:49]  INFO task qps: 10000, num: 100000, timeNeed: 0.000s
[2020-03-31 15:58:49]  INFO rate: 6698910.757
[2020-03-31 15:58:49]  INFO rate: 6132110.182
[2020-03-31 15:58:49]  INFO rate: 6394802.305
[2020-03-31 15:58:49]  INFO rate: 6386511.687
[2020-03-31 15:58:49]  INFO rate: 5484591.041
[2020-03-31 15:58:49]  INFO rate: 6646770.666
[2020-03-31 15:58:49]  INFO rate: 5725671.621
[2020-03-31 15:58:49]  INFO rate: 6449490.813
[2020-03-31 15:58:49]  INFO rate: 5718631.875
[2020-03-31 15:58:49]  INFO rate: 6620981.892
[2020-03-31 15:58:49]  INFO rate: 10174.362
[2020-03-31 15:58:49]  INFO rate: 9999.993
[2020-03-31 15:58:49]  INFO rate: 9999.979
[2020-03-31 15:58:49]  INFO rate: 10000.032
[2020-03-31 15:58:49]  INFO rate: 9999.993
[2020-03-31 15:58:49]  INFO rate: 10000.001
[2020-03-31 15:58:50]  INFO rate: 9999.981
[2020-03-31 15:58:50]  INFO rate: 9750.039
[2020-03-31 15:58:50]  INFO rate: 10263.124
[2020-03-31 15:58:50]  INFO rate: 10000.008
[2020-03-31 15:58:50]  INFO rate: 10000.004
[2020-03-31 15:58:50]  INFO rate: 10000.006
[2020-03-31 15:58:50]  INFO rate: 9999.980
[2020-03-31 15:58:50]  INFO rate: 10000.013
[2020-03-31 15:58:50]  INFO rate: 10000.008
[2020-03-31 15:58:50]  INFO rate: 9999.989
[2020-03-31 15:58:51]  INFO rate: 10000.006
[2020-03-31 15:58:51]  INFO rate: 10000.002
[2020-03-31 15:58:51]  INFO rate: 10000.002
[2020-03-31 15:58:51]  INFO rate: 9999.990
[2020-03-31 15:58:51]  INFO rate: 9999.990
[2020-03-31 15:58:51]  INFO rate: 10000.004
[2020-03-31 15:58:51]  INFO rate: 10000.008
[2020-03-31 15:58:51]  INFO rate: 10000.004
[2020-03-31 15:58:51]  INFO rate: 9999.999
[2020-03-31 15:58:51]  INFO rate: 9999.994
[2020-03-31 15:58:52]  INFO rate: 10000.011
[2020-03-31 15:58:52]  INFO rate: 9999.991
[2020-03-31 15:58:52]  INFO rate: 9865.780
[2020-03-31 15:58:52]  INFO rate: 10137.923
[2020-03-31 15:58:52]  INFO rate: 9999.998
[2020-03-31 15:58:52]  INFO rate: 10000.009
[2020-03-31 15:58:52]  INFO rate: 10000.000
[2020-03-31 15:58:52]  INFO rate: 9999.998
[2020-03-31 15:58:52]  INFO rate: 10000.000
[2020-03-31 15:58:52]  INFO rate: 10000.000
[2020-03-31 15:58:53]  INFO rate: 10000.002
[2020-03-31 15:58:53]  INFO rate: 10000.006
[2020-03-31 15:58:53]  INFO rate: 9999.994
[2020-03-31 15:58:53]  INFO rate: 9999.992
[2020-03-31 15:58:53]  INFO rate: 9999.989
[2020-03-31 15:58:53]  INFO rate: 10000.026
[2020-03-31 15:58:53]  INFO rate: 9999.996
[2020-03-31 15:58:53]  INFO rate: 10000.001
[2020-03-31 15:58:53]  INFO rate: 9999.975
[2020-03-31 15:58:53]  INFO rate: 10000.015
[2020-03-31 15:58:54]  INFO rate: 10000.008
[2020-03-31 15:58:54]  INFO rate: 9999.993
[2020-03-31 15:58:54]  INFO rate: 10000.011
[2020-03-31 15:58:54]  INFO rate: 9998.373
[2020-03-31 15:58:54]  INFO rate: 10001.612
[2020-03-31 15:58:54]  INFO rate: 10000.003
[2020-03-31 15:58:54]  INFO rate: 10000.010
[2020-03-31 15:58:54]  INFO rate: 10000.000
[2020-03-31 15:58:54]  INFO rate: 10000.000
[2020-03-31 15:58:54]  INFO rate: 9999.988
[2020-03-31 15:58:55]  INFO rate: 10000.005
[2020-03-31 15:58:55]  INFO rate: 10000.007
[2020-03-31 15:58:55]  INFO rate: 9999.996
[2020-03-31 15:58:55]  INFO rate: 9999.999
[2020-03-31 15:58:55]  INFO rate: 9999.976
[2020-03-31 15:58:55]  INFO rate: 10000.032
[2020-03-31 15:58:55]  INFO rate: 9999.998
[2020-03-31 15:58:55]  INFO rate: 10000.000
[2020-03-31 15:58:55]  INFO rate: 9999.990
[2020-03-31 15:58:55]  INFO rate: 10000.001
[2020-03-31 15:58:56]  INFO rate: 9999.999
[2020-03-31 15:58:56]  INFO rate: 9999.993
[2020-03-31 15:58:56]  INFO rate: 10000.012
[2020-03-31 15:58:56]  INFO rate: 10000.001
[2020-03-31 15:58:56]  INFO rate: 10000.005
[2020-03-31 15:58:56]  INFO rate: 9999.996
[2020-03-31 15:58:56]  INFO rate: 9999.993
[2020-03-31 15:58:56]  INFO rate: 9999.999
[2020-03-31 15:58:56]  INFO rate: 10000.008
[2020-03-31 15:58:56]  INFO rate: 9999.996
[2020-03-31 15:58:57]  INFO rate: 9998.998
[2020-03-31 15:58:57]  INFO rate: 10001.010
[2020-03-31 15:58:57]  INFO rate: 9999.992
[2020-03-31 15:58:57]  INFO rate: 10000.004
[2020-03-31 15:58:57]  INFO rate: 9999.997
[2020-03-31 15:58:57]  INFO rate: 9999.988
[2020-03-31 15:58:57]  INFO rate: 10000.012
[2020-03-31 15:58:57]  INFO rate: 9999.996
[2020-03-31 15:58:57]  INFO rate: 10000.012
[2020-03-31 15:58:57]  INFO rate: 9999.998
[2020-03-31 15:58:58]  INFO rate: 9999.991
[2020-03-31 15:58:58]  INFO rate: 9999.990
[2020-03-31 15:58:58]  INFO rate: 10000.020
[2020-03-31 15:58:58]  INFO rate: 9999.995
[2020-03-31 15:58:58]  INFO task over, used: 9.000s
[2020-03-31 15:58:58]  INFO task qps: 10000, num: 15000, timeNeed: 0.002s
[2020-03-31 15:59:00]  INFO rate: 397.098
[2020-03-31 15:59:03]  INFO rate: 396.814
[2020-03-31 15:59:05]  INFO rate: 397.583
[2020-03-31 15:59:08]  INFO rate: 391.641
[2020-03-31 15:59:10]  INFO rate: 391.799
[2020-03-31 15:59:13]  INFO rate: 400.202
[2020-03-31 15:59:15]  INFO rate: 402.581
[2020-03-31 15:59:18]  INFO rate: 405.188
[2020-03-31 15:59:20]  INFO rate: 396.681
[2020-03-31 15:59:23]  INFO rate: 397.131
[2020-03-31 15:59:25]  INFO rate: 398.076
[2020-03-31 15:59:28]  INFO rate: 398.811
[2020-03-31 15:59:30]  INFO rate: 399.199
[2020-03-31 15:59:33]  INFO rate: 400.549
[2020-03-31 15:59:36]  INFO rate: 390.775
[2020-03-31 15:59:36]  INFO task over, used: 37.732s
[2020-03-31 15:59:36]  INFO task qps: 500, num: 10000, timeNeed: 0.000s
[2020-03-31 15:59:37]  INFO rate: 1002.004
[2020-03-31 15:59:39]  INFO rate: 500.000
[2020-03-31 15:59:41]  INFO rate: 500.000
[2020-03-31 15:59:43]  INFO rate: 500.000
[2020-03-31 15:59:45]  INFO rate: 500.000
[2020-03-31 15:59:47]  INFO rate: 500.000
[2020-03-31 15:59:49]  INFO rate: 500.000
[2020-03-31 15:59:51]  INFO rate: 500.000
[2020-03-31 15:59:53]  INFO rate: 500.000
[2020-03-31 15:59:55]  INFO rate: 500.000
[2020-03-31 15:59:55]  INFO task over, used: 19.000s
```