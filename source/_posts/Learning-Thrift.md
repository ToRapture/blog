---
title: Learning Thrift
date: 2020-05-27 17:39:00
tags:
- Thrift
---

## IDL and Code Generation
`easy.thrift`:
```thrift
namespace go easy
namespace py easy
namespace cpp easy


service Easy {
    void ping();
    string say_hello(1: list<string> words);
    i32 add(1: i32 x, 2: i32 y);
}

service Hard {
    void ping();
    string say_hello(1: list<string> words);
    i32 add(1: i32 x, 2: i32 y);
}

```
`$ thrift -r --gen go ./easy.thrift`

## Golang

`goland`:
```go
package main

import (
	"context"
	"errors"
	"math/rand"
	"os"
	"strings"
	"time"

	"go-learn/gen-go/easy"

	"github.com/apache/thrift/lib/go/thrift"
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

type EasyHandler struct{}

func (h *EasyHandler) Ping(ctx context.Context) error {
	logger.Infof("ping")
	if rand.Int()%2 == 0 {
		logger.Infof("return error")
		return errors.New("a random error")
	}
	return nil
}

func (h *EasyHandler) SayHello(ctx context.Context, words []string) (string, error) {
	return strings.Join(words, ", "), nil
}

func (h *EasyHandler) Add(ctx context.Context, x int32, y int32) (int32, error) {
	return x + y, nil
}

func runEasy() error {
	processor := easy.NewEasyProcessor(&EasyHandler{})
	transport, err := thrift.NewTServerSocket(":27024")
	if err != nil {
		return err
	}
	transportFactory := thrift.NewTTransportFactory()
	protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()

	server := thrift.NewTSimpleServer4(processor, transport, transportFactory, protocolFactory)
	return server.Serve()
}

func runClient() error {
	var transport thrift.TTransport
	transport, err := thrift.NewTSocket(":27024")
	if err != nil {
		return err
	}
	transportFactory := thrift.NewTTransportFactory()
	protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()
	transport, err = transportFactory.GetTransport(transport)
	if err != nil {
		return err
	}

	defer transport.Close()
	if err := transport.Open(); err != nil {
		return err
	}

	iprot := protocolFactory.GetProtocol(transport)
	oprot := protocolFactory.GetProtocol(transport)
	client := easy.NewEasyClient(thrift.NewTStandardClient(iprot, oprot))

	if err := client.Ping(context.Background()); err != nil {
		logger.Errorf("ping error: %v", err)
	}
	if sum, err := client.Add(context.Background(), 1, 2); err != nil {
		logger.Errorf("add error: %v", err)
	} else {
		logger.Infof("result of add: %v", sum)
	}
	if ret, err := client.SayHello(context.Background(), []string{"hello", "world"}); err != nil {
		logger.Errorf("say hello error: %v", err)
	} else {
		logger.Infof("result of say hello: %v", ret)
	}

	return nil
}

func main() {
	go func() {
		err := runEasy()
		if err != nil {
			logger.Errorf("run server error: %v", err)
		}
	}()
	time.Sleep(time.Second * 2)
	go func() {
		err := runClient()
		if err != nil {
			logger.Errorf("run client error: %v", err)
		}
	}()
	time.Sleep(time.Second * 2)
}

```

## Python

`python-server`:
```python
# coding: utf-8

from easy.easy import Easy


from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol
from thrift.server import TServer


class EasyHandler:
    def __init__(self):
        pass

    @staticmethod
    def ping():
        print("ping")

    @staticmethod
    def say_hello(words):
        return ", ".join(words)

    @staticmethod
    def add(x, y):
        return x + y


if __name__ == '__main__':
    handler = EasyHandler()
    processor = Easy.Processor(handler)
    transport = TSocket.TServerSocket(host='127.0.0.1', port=9090)
    tfactory = TTransport.TBufferedTransportFactory()
    pfactory = TBinaryProtocol.TBinaryProtocolFactory()

    server = TServer.TSimpleServer(processor, transport, tfactory, pfactory)
    server.serve()

```

`python-client`:
```python
# coding: utf-8

from thrift.protocol import TBinaryProtocol
from thrift.transport import TSocket
from thrift.transport import TTransport

from easy.easy import Easy


def main():
    transport = TSocket.TSocket('localhost', 9090)
    transport = TTransport.TBufferedTransport(transport)
    protocol = TBinaryProtocol.TBinaryProtocol(transport)

    client = Easy.Client(protocol)

    transport.open()
    print(client.ping())
    print(client.add(1, 2))
    print(client.say_hello(["hello", "world"]))
    transport.close()


if __name__ == '__main__':
    main()

```