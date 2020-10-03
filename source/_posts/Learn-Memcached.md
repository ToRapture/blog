---
title: Learn Memcached
date: 2020-05-19 18:33:00
tags:
- Memcached
---

# Deployment
https://github.com/memcached/memcached
See the file `BUILD` to learn how to build.

The default configuration file `/etc/memcached.conf` is used by `scripts/start-memcached` when using this script to start service.

# Protocol
Protocol is described in `doc/protocol.txt`.

`server`:
```
$ memcached -vv -m 3 -M
<17 server listening (auto-negotiate)
<18 server listening (auto-negotiate)
<19 new auto-negotiating client connection
19: Client using the ascii protocol
<19 get foo
>19 END
<19 set foo 0 0 3
>19 STORED
<19 get foo
>19 sending key foo
>19 END
```

`client`:
```
$ nc -c localhost 11211
get foo
END
set foo 0 0 3
bar
STORED
get foo
VALUE foo 0 3
bar
END
```

# Evication
`server`:
```
$ memcached -vv -m 3 -M
<19 new auto-negotiating client connection
19: Client using the ascii protocol
<19 set 0 0 0 10240
>19 STORED
<19 set 1 0 0 10240
>19 STORED
<19 set 2 0 0 10240
>19 STORED
<19 set 3 0 0 10240
>19 STORED
<19 set 4 0 0 10240
>19 STORED
<19 set 5 0 0 10240
>19 STORED
<19 set 6 0 0 10240
>19 STORED
......
<19 set 280 0 0 10240
>19 STORED
<19 set 281 0 0 10240
>19 STORED
<19 set 282 0 0 10240
>19 SERVER_ERROR out of memory storing object
<19 connection closed.
```

`client`:
```python
# coding: utf-8

from pymemcache.client import base


def main():
    client = base.Client(('localhost', 11211), default_noreply=False)
    sz = 0
    pos = 0
    while True:
        key = str(pos)
        value = 'x' * 10240
        try:
            res = client.set(key, value)
            if res:
                sz += len(key) + len(value)
                pos += 1
            else:
                break
        except Exception as e:
            print('exception: %s, sz: %s' % (e, sz))
            break


if __name__ == "__main__":
    main()

```
```
$ python3 main.py
exception: b'out of memory storing object', sz: 2888416
```

```go
package main

import (
	"bytes"
	"os"

	"github.com/bradfitz/gomemcache/memcache"
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
	mc := memcache.New("localhost:11211")

	buf := bytes.NewBuffer(nil)
	for i := 0; i < 10240; i++ {
		buf.WriteRune('x')
	}

	if err := mc.Set(&memcache.Item{
		Key:   "foo",
		Value: buf.Bytes(),
	}); err != nil {
		logger.Errorf("set error: %v", err)
	}

	if it, err := mc.Get("32"); err != nil {
		logger.Errorf("get error: %v, is miss: %v", err, err == memcache.ErrCacheMiss)
	} else {
		logger.Infof("key: %v, value: %v", it.Key, string(it.Value))
	}

	if it, err := mc.Get("non"); err != nil {
		logger.Errorf("get error: %v, is miss: %v", err, err == memcache.ErrCacheMiss)
	} else {
		logger.Infof("key: %v, value: %v", it.Key, string(it.Value))
	}
}
```

```
[2020-05-19 18:21:20] ERROR set error: memcache: unexpected response line from "set": "SERVER_ERROR out of memory storing object\r\n"
[2020-05-19 18:21:20]  INFO key: 32, value: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
[2020-05-19 18:21:20] ERROR get error: memcache: cache miss, is miss: true
```