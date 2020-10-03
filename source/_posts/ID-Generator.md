---
title: ID Generator
date: 2020-04-01 16:52:00
tags:
- Design
---

## Implementation
Some details of this implementation borrow from [sonyflake](https://github.com/sony/sonyflake).
Here using 8 bits to represent time in units of 500ms, 4 bits to represent sequence, 4 bits to represent machine.
As a result:
- The lifetime is $2^8 \times 500ms = 128s$
- It can work in $2^4 = 16$ distributed machines
- It can generate $2^4 = 16$ IDs per $500ms$ at most in a single thread

### Code
`server`:
```go
package main

import (
	"errors"
	"net/http"
	"os"
	"sync"
	"time"

	"github.com/gin-gonic/gin"
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

const (
	BitLenTick      = 8
	BitLenSequence  = 4
	BitLenMachineID = 4

	tickTimeUnit = time.Millisecond * 500
)

var idFlake *IDFlake

type IDFlake struct {
	mu sync.Mutex

	machineID   int64
	startTick   int64 // ticks of the start time of IDFlake from UNIX epoch
	elapsedTick int64 // elapsed ticks from start tick
	sequence    int64
}

func toFlakeTime(t time.Time) int64 {
	return t.UTC().UnixNano() / int64(tickTimeUnit)
}

func NewIDFlake(machineID int64, startTime time.Time) *IDFlake {
	return &IDFlake{
		machineID:   machineID,
		startTick:   toFlakeTime(startTime),
		elapsedTick: 0,
		sequence:    0,
	}
}

func (f *IDFlake) toID() (uint64, error) {
	if f.elapsedTick >= 1<<BitLenTick {
		return 0, errors.New("over the time limit")
	}

	return uint64(f.elapsedTick<<(BitLenMachineID+BitLenSequence)) | uint64(f.machineID<<BitLenSequence) | uint64(f.sequence), nil
}

func (f *IDFlake) getID() (uint64, error) {
	f.mu.Lock()
	defer f.mu.Unlock()

	currentTick := toFlakeTime(time.Now()) - f.startTick
	if f.elapsedTick < currentTick {
		f.sequence = 0
		f.elapsedTick = currentTick
	} else {
		f.sequence = (f.sequence + 1) & (1<<BitLenSequence - 1)
		if f.sequence == 0 {
			f.elapsedTick++
			sleepFor := sleepTime(f.elapsedTick - currentTick)
			time.Sleep(sleepFor)
			logger.Infof("slept for %.8fms", sleepFor.Seconds()*1000)
		}
	}

	return f.toID()
}

func sleepTime(overTick int64) time.Duration {
	return time.Duration(overTick)*tickTimeUnit - time.Duration(time.Now().UTC().UnixNano())%tickTimeUnit
}

func Decompose(id uint64) map[string]uint64 {
	return map[string]uint64{
		"id":       id,
		"tick":     id >> (BitLenMachineID + BitLenSequence),
		"machine":  (id >> BitLenSequence) & (1<<BitLenMachineID - 1),
		"sequence": id & (1<<BitLenSequence - 1),
	}
}

func getID(c *gin.Context) {
	id, err := idFlake.getID()
	if err != nil {
		logger.Warnf("getID error: %v", err)
		c.JSON(http.StatusOK,
			map[string]interface{}{
				"error": err.Error(),
			},
		)
		return
	}
	result := Decompose(id)
	c.JSON(http.StatusOK, result)
}

func httpRun() {
	r := gin.Default()
	r.GET("/get_id", getID)
	r.Run(":8080")
}

func main() {
	idFlake = NewIDFlake(8, time.Now())
	httpRun()
}
```

`client`:
```python
# coding: utf-8

import requests


def main():
    while True:
        res = requests.get('http://127.0.0.1:8080/get_id').json()
        print res
        if res.get('error'):
            break


if __name__ == '__main__':
    main()
```

### Run
`server`:
```
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] Listening and serving HTTP on :8080
[2020-04-01 16:11:05]  INFO slept for 285.71300000ms
[2020-04-01 16:11:06]  INFO slept for 446.46500000ms
[2020-04-01 16:11:06]  INFO slept for 450.56500000ms
[2020-04-01 16:11:07]  INFO slept for 449.93100000ms
......
[2020-04-01 16:13:08]  INFO slept for 449.52100000ms
[2020-04-01 16:13:09]  INFO slept for 441.43400000ms
[2020-04-01 16:13:09]  INFO slept for 434.80600000ms
[2020-04-01 16:13:10]  INFO slept for 445.62500000ms
[2020-04-01 16:13:10]  INFO slept for 444.83400000ms
[2020-04-01 16:13:11]  INFO slept for 437.82700000ms
[2020-04-01 16:13:11]  WARN getID error: over the time limit
```

`client`:
```
{u'machine': 0, u'tick': 4, u'id': 1024, u'sequence': 0}
{u'machine': 0, u'tick': 4, u'id': 1025, u'sequence': 1}
{u'machine': 0, u'tick': 4, u'id': 1026, u'sequence': 2}
{u'machine': 0, u'tick': 4, u'id': 1027, u'sequence': 3}
{u'machine': 0, u'tick': 4, u'id': 1028, u'sequence': 4}
{u'machine': 0, u'tick': 4, u'id': 1029, u'sequence': 5}
{u'machine': 0, u'tick': 4, u'id': 1030, u'sequence': 6}
{u'machine': 0, u'tick': 4, u'id': 1031, u'sequence': 7}
{u'machine': 0, u'tick': 4, u'id': 1032, u'sequence': 8}
{u'machine': 0, u'tick': 4, u'id': 1033, u'sequence': 9}
{u'machine': 0, u'tick': 4, u'id': 1034, u'sequence': 10}
{u'machine': 0, u'tick': 4, u'id': 1035, u'sequence': 11}
{u'machine': 0, u'tick': 4, u'id': 1036, u'sequence': 12}
{u'machine': 0, u'tick': 4, u'id': 1037, u'sequence': 13}
{u'machine': 0, u'tick': 4, u'id': 1038, u'sequence': 14}
{u'machine': 0, u'tick': 4, u'id': 1039, u'sequence': 15}
{u'machine': 0, u'tick': 5, u'id': 1280, u'sequence': 0}
{u'machine': 0, u'tick': 5, u'id': 1281, u'sequence': 1}
{u'machine': 0, u'tick': 5, u'id': 1282, u'sequence': 2}
{u'machine': 0, u'tick': 5, u'id': 1283, u'sequence': 3}
{u'machine': 0, u'tick': 5, u'id': 1284, u'sequence': 4}
......
{u'machine': 0, u'tick': 254, u'id': 65037, u'sequence': 13}
{u'machine': 0, u'tick': 254, u'id': 65038, u'sequence': 14}
{u'machine': 0, u'tick': 254, u'id': 65039, u'sequence': 15}
{u'machine': 0, u'tick': 255, u'id': 65280, u'sequence': 0}
{u'machine': 0, u'tick': 255, u'id': 65281, u'sequence': 1}
{u'machine': 0, u'tick': 255, u'id': 65282, u'sequence': 2}
{u'machine': 0, u'tick': 255, u'id': 65283, u'sequence': 3}
{u'machine': 0, u'tick': 255, u'id': 65284, u'sequence': 4}
{u'machine': 0, u'tick': 255, u'id': 65285, u'sequence': 5}
{u'machine': 0, u'tick': 255, u'id': 65286, u'sequence': 6}
{u'machine': 0, u'tick': 255, u'id': 65287, u'sequence': 7}
{u'machine': 0, u'tick': 255, u'id': 65288, u'sequence': 8}
{u'machine': 0, u'tick': 255, u'id': 65289, u'sequence': 9}
{u'machine': 0, u'tick': 255, u'id': 65290, u'sequence': 10}
{u'machine': 0, u'tick': 255, u'id': 65291, u'sequence': 11}
{u'machine': 0, u'tick': 255, u'id': 65292, u'sequence': 12}
{u'machine': 0, u'tick': 255, u'id': 65293, u'sequence': 13}
{u'machine': 0, u'tick': 255, u'id': 65294, u'sequence': 14}
{u'machine': 0, u'tick': 255, u'id': 65295, u'sequence': 15}
{u'error': u'over the time limit'}
```