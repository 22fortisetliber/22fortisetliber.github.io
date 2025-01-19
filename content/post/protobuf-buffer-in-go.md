+++
title = "Protocol Buffer in Go"
date = "2025-01-16"
author = "Son Vu Thai"
tags = [
    "Golang",
    "Programing Language"
]
+++

# What is Protocol Buffer

According to [offcial documentation](https://protobuf.dev/):

> Protocol buffers are Google’s language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use specially generated source code to easily write and read your structured data to and from various data streams and using a variety of languages.

Well, let's understand this with a few examples!
Imagine you store data about books, so a sample of XML will look like this:

```xml
<book>
	<name>Animal Farm</name>
	<author>George Orwell</author>
	<score>10</score>
</book>
```

in JSON, it will look like this:

```json
{
  "name": "Animal Farm",
  "author": "George Orwell",
  "score": 10
}
```

OK, let's convert it into protocol buffers

```Golang

package main

import (
	"fmt"
	"os"

	"github.com/22fortisetliber/lab/protobuf/pb"
	"google.golang.org/protobuf/encoding/protojson"
	"google.golang.org/protobuf/proto"
)

func main() {
	data, err := os.ReadFile("data.json")
	if err != nil {
		fmt.Println("err on reading file", err)
		return
	}
	fmt.Println(data)
	bpb := &pb.Book{}
	err = protojson.Unmarshal(data, bpb)
	if err != nil {
		fmt.Println("failed to unmarshal", err)
		return
	}
	bytes, err := proto.Marshal(bpb)
	if err != nil {
		fmt.Println("failed to marshal", err)
		return
	}
	fmt.Println(bytes)
}

```

and boom, we got:

```bash
[10 11 65 110 105 109 97 108 32 70 97 114 109 18 13 71 101 111 114 103 101 32 79 114 119 101 108 108 24 10]
```

If you observe the wire encoded closely, you might see the book's name, "Animal Farm" is spelled out with "A" = 65 in position 2 of the array and so on. The last array element is a byte representation of the score (10). If you like to dive into how protobuf encodes data to file or to the wire, you can read more in [here](https://protobuf.dev/programming-guides/encoding/) or just wait for my next post :wink:

In this example, the size of both JSON and Protocol Buffer seems no different. But as your data increases, a lot of size and complexity gets removed with the result of smaller and more efficient payloads for application.
