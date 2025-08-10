+++
title = "Golang 101: Some quick notes about channel"
date = "2025-08-09"
author = "Son Vu Thai"
tags = [
    "Golang",
    "Golang 101",
    "Programing Language"
]
+++

# Table of contents

1. [Channels](#i-channels)
   - 1.1. [Creating and Using Channels](#11-creating-and-using-channels)
   - 1.2. [Channel Direction](#12-channel-direction)
   - 1.3. [Buffered Channels](#13-buffered-channels)
   - 1.4. [Rob Pike's Channel Patterns](#14-rob-pikes-channel-patterns)
     - 1.4.1. [The Generator Pattern](#141-the-generator-pattern)
     - 1.4.2. [Channel as a Service](#142-channel-as-a-service)
     - 1.4.3. [Closing Channel](#143-closing-channel)
     - 1.4.4. [Range Over Channels](#144-range-over-channels)
   - 1.5. [Common Pitfalls](#15-common-pitfalls)
     - 1.5.1. [Deadlock](#151-deadlock)
     - 1.5.2. [Goroutine leak](#152-goroutine-leak)

# I. Channels

Channels are typed conduits through which you can send and receive values using the channel operator ==&larr;== .

## 1.1. Creating and Using Channels

**Basic Channel Operations**

```Golang
package main

import "fmt"

func main() {
    // Create a channel of strings
    messages := make(chan string)

    // Send a value in a goroutine
    go func() {
        messages <- "ping"
    }()

    // Receive the value in main
    msg := <-messages
    fmt.Println(msg) // "ping"
}
```

**Key Points:**

- Channels must be created before use with make(chan Type)
- Sends and receives block by default
- Blocking ensures synchronization between goroutines

## 1.2. Channel Direction

Channels can be derection when used as function parameters:

```Golang
// Send only channel
func send(ch chan<- string) {
    ch <- "hello"
}

// Receive only channel
func receive(ch <-chan string){
    msg := <-ch
    fmt.Println(msg)
}
```

## 1.3. Buffered Channels

By default, channels are unbuffered - they only accept sends when there's a corresponding receive ready. Buffered channels accept a limited number of values without a receiver:

```Golang
func main() {
    // Create a buffered channel with capacity 2
    ch := make(chan int, 2)

    // These sends won't block
    ch <- 1
    ch <- 2

    // This would block: ch <- 3

    // Receive values
    fmt.Println(<-ch) // 1
    fmt.Println(<-ch) // 2
}
```

| Aspect          | Unbuffered           | Buffered                |
| --------------- | -------------------- | ----------------------- |
| Creation        | make(chan T)         | make(chan T, n)         |
| Send blocks     | Until receiver ready | When buffer full        |
| Receive blocks  | Until sender ready   | When buffer empty       |
| Synchronization | Guaranteed           | Looser coupling         |
| Use case        | Synchronization      | Performance, decoupling |

## 1.4. Rob Pike's Channel Patterns

### 1.4.1. The Generator Pattern

From Rob Pike's talks, a generator is a function that returns a channel:

```Golang
func boring(msg string) <-chan string {
    c := make(chan string)
    go func() {
        for i := 0; ; i++ {
            c <- fmt.Sprintf("%s %d", msg, i)
            time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
        }
    }()
    return c
}

func main() {
    c := boring("boring!")
    for i := 0; i < 5; i++ {
        fmt.Printf("You say: %q\n", <-c)
    }
    fmt.Println("You're boring; I'm leaving.")
}
```

### 1.4.2. Channel as a Service

```Golang
type Request struct {
    args        []int
    resultChan  chan int
}

func sum(a []int) int {
    s := 0
    for _, v := range a {
        s += v
    }
    return s
}

func server(requests chan *Request) {
    for req := range requests {
        go func(req *Request) {
            req.resultChan <- sum(req.args)
        }(req)
    }
}

func main() {
    requests := make(chan *Request)
    go server(requests)

    req := &Request {
        args: []int{1,2,3},
        resultChan: make(chan int),
    }
    requests <- req
    result := <- req.resultChan
    fmt.Println("Sum: ", result)
}
```

This demonstrates how channels can represent as a service, where:

- Request struct: Represents as a service request with:

  - args []int: Input data for service
  - resultChan chan int: Private channel for receiving the result

- server() function:
  - Listen on a ==requests channel== for incoming request
  - Spawns a goroutine for each request
  - Send result back through each request's private result channel

Some key benefits:

- **Service implement is hidden behind channel interface**
- **Concurrently process multiple requests**
- **Client gets result exactly when it ready**
- **Data of each request is isolated**

### 1.4.3. Closing Channel

A sender can close a channel to indicate no more values will be sent:

```Golang
func main() {
    jobs := make(chan int, 5)
    done := make(chan bool)

    go func() {
        for {
            j, more := <-jobs
            if !more {
                fmt.Println("received all jobs")
                done <- true
                return
            }
            fmt.Println("received job", j)
        }
    }()

    for j := 1; j <= 3; j++ {
        jobs <- j
        fmt.Println("sent job", j)
    }
    close(jobs)
    fmt.Println("sent all jobs")

    <-done
}
```

**Rules for Closing:**

- Only the ==sender== should close a channel
- Sending on a closed channel causes panic
- Receiving from a closed channel returns zero value
- Use v, ok := <-ch to check if channel is closed

## 1.4.4. Range Over Channels

You can use range to receive values until a channel is closed:

```Golang
func main() {
    queue := make(chan string, 2)
    queue <- "one"
    queue <- "two"
    close(queue)

    for elem := range queue {
        fmt.Println(elem)
    }
}
```

## 1.5. Common Pitfalls

### 1.5.1. Deadlock

```Golang
// Deadlock - all goroutines blocked
func main() {
    ch := make(chan int)
    ch <- 42  // Blocks forever, no receiver
    fmt.Println(<-ch)
}
```

**Explain:**

- ==ch := make(chan int)== create an unbuffered int channel
- ==ch <- 42== this code attempt to send value (42), but block because there is no receiver ready
- ==fmt.Println(<-ch)== This line would receive the value 42, but it is never reached because promgram stuck on the previous line
- That's happend because all of the logic (unbuffered channel, send, receive) running on only one goroutine

**&rarr; To fix this problem, we can use other goroutine, or buffered channel**

### 1.5.2. Goroutine leak

```Golang
// Leak - goroutine runs forever
func leak() {
    ch := make(chan int)
    go func() {
        val := <-ch  // Blocks forever if ch never receives
        fmt.Println(val)
    }()
    // Function returns, goroutine still running
}
```

**Explain:**
Even this function return, the goroutine still running because nothing ever send to the channel.

### 1.5.3. Race on Channel Variable

```Golang
// Race condition
var ch chan int

func main() {
    go func() {
        ch = make(chan int)  // Goroutine 1, Race!
    }()
    go func() {
        ch <- 42  // Goroutine 2, Race!
    }()
}
```

**Explain:**
After initial nil channel ==ch== on main(), 2 concurrent goroutine try to access this channel (goroutine 1 tries to create channel, goroutine 2 tries to send value 42 to channel, which leads to an undefined behavior due ti concurent read/write of the same memory location).
