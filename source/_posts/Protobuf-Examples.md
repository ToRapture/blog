---
title: Protobuf Examples
date: 2020-06-07 13:37:00
tags: Protobuf
---

# Installation
https://github.com/protocolbuffers/protobuf/blob/master/src/README.md

# Proto
`pb/entities.proto`:
```protobuf
syntax = "proto3";

package entities;
option go_package = "pb/entities";

enum Gender {
    UNKNOWN = 0;
    MALE = 1;
    FEMALE = 2;
}

message Person {
    Gender gender = 1;
    string name = 2;
    repeated string tags = 3;
    double height = 4;
    int32 age = 5;
    oneof important {
        string task = 6;
        int32 problem = 7;
        double point = 8;
    }
    map<string, string> extra = 10;
}

```

# Golang
https://developers.google.com/protocol-buffers/docs/gotutorial

`$ protoc --proto_path=./ --go_out=./ pb/entities.proto`

```go
package main

import (
	"io/ioutil"
	"os"

	"./pb/entities"
	"github.com/golang/protobuf/proto"
	logger "github.com/sirupsen/logrus"
	prefixed "github.com/x-cray/logrus-prefixed-formatter"
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
	p := &entities.Person{
		Gender:    entities.Gender_FEMALE,
		Name:      "ToRapture",
		Tags:      []string{"Hello", "World"},
		Height:    114,
		Age:       32,
		Important: &entities.Person_Task{Task: "OneOfExample"},
		Extra: map[string]string{
			"who":   "are you",
			"where": "to go",
		},
	}

	logger.Infof("p: %v", p)
	bt, err := proto.Marshal(p)
	if err != nil {
		logger.Errorf("proto marshal error: %v", err)
		return
	}

	b := new(entities.Person)
	if err = proto.Unmarshal(bt, b); err != nil {
		logger.Errorf("proto unmarshal error: %v", err)
		return
	}

	logger.Infof("b: %v, b == p: %v", b, proto.Equal(b, p))
	switch oneOf := b.GetImportant().(type) {
	case *entities.Person_Task:
		logger.Infof("oneOf is task, value: %v", oneOf.Task)
	case *entities.Person_Point:
		logger.Infof("oneOf is point value: %v", oneOf.Point)
	case *entities.Person_Problem:
		logger.Infof("oneOf is problem value: %v", oneOf.Problem)
	}

	if err := ioutil.WriteFile("person.data", bt, 0666); err != nil {
		logger.Errorf("write to file error: %v", err)
		return
	}
}

```

# Python
https://developers.google.com/protocol-buffers/docs/pythontutorial

`$ protoc --proto_path=./ --python_out=../python pb/entities.proto`

```python
# coding: utf-8

from pb import entities_pb2


def main():
    data = b""
    with open("../go/person.data", "rb") as f:
        data = f.read()
    person = entities_pb2.Person()
    person.name = "ToRapture"
    print("person: %s" % person)
    print("serialized: %s" % person.SerializeToString())

    person.ParseFromString(data)
    print("parsed: %s" % person)

    which = person.WhichOneof('important')
    print('which: %s' % which)
    if which:
        print('oneof value: %s' % getattr(person, which))


if __name__ == '__main__':
    main()

```