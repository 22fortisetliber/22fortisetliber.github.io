+++
title = "Golang 101: Some quick notes"
date = "2025-08-09"
author = "Son Vu Thai"
tags = [
    "Golang",
    "Programing Language"
]
+++

# Table of contents

1. [Goroutines](#goroutines)
   1. [Goroutine Lifecycle](#goroutine-lifecycle)
   2. [Program Termination](#program-termination)
   3. [Summary](#summary)
2. [Channels](#channels)
   1. [Creating and Using Channels](#creating-and-using-channels)
   2. [Channel Direction](#channel-direction)
   3. [Buffered Channels](#buffered-channels)

# Goroutines

Goroutines are lightweight threads managed by the Go runtime. Each one is a function that runs concurrently with the other in the same address space.

**Key Characteristics**

- Lightweight: Only 2KB of stack space initially
- Growable: Stack grows and shrinks as needed
- Multiplexed: Many goroutines run on few OS threads
- Cheap: Creating millions is feasible

Let's tweak Rob Pike's famous example:

```Golang
package main

import (
    "fmt"
    "time"
)

func boring(msg string) {
    for i := 0; ; i++ {
        fmt.Printf("%s %d\n", msg, i)
        time.Sleep(time.Second)
    }
}

func main() {
    go boring("boring!")
    fmt.Println("I'm listening.")
    time.Sleep(2 * time.Second)
    fmt.Println("You're boring; I'm leaving.")
}
```

**Console Output:**

```Console
23:21:26 I'm listening.
23:21:26 boring 0
23:21:27 boring 1
23:21:28 You're boring; I'm leaving.
```

What happens when running this program:

1. main() start (goroutine 0)
2. go boring("boring!") creates goroutine 1
3. main prints "I'm listening." and both goroutines run concurrently
4. Main sleeps for 2 seconds
5. While main is sleeping, goroutine 1 (boring) print boring 0
6. While main is sleeping, goroutine 1 (boring) print boring 1
7. Main prints "You're boring; I'm leaving." after sleeps 2 seconds
8. Main exits, terminating the entire program

## Goroutine Lifecycle

1. Created - goroutine is spawned
2. Running - actively executing
3. Sleeping - time.Sleep() or waiting
4. Terminated - function returns

## Program Termination

Critical Point: When main() returns, the entire program exits, even if other goroutines are still running.

## Summary

- Goroutines are lightweight and cheap
- Use go keyword to launch them
- They run concurrently, not in sequence
- Program exits when main() returns
- Perfect for I/O-bound tasks

# Channels

Channels are typed conduits through which you can send and receive values using the channel operator **&larr;** .

## Creating and Using Channels

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

## Channel Direction

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

## Buffered Channels

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

## Rob Pike's Channel Patterns

### The Generator Pattern

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

So, what happens when running this program:
