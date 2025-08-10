+++
title = "Golang 101: Some quick notes about goroutine"
date = "2025-08-10"
author = "Son Vu Thai"
tags = [
    "Golang",
    "Golang 101",
    "Programing Language"
]
+++

# Table of contents

1. [Goroutines](#i-goroutines)
   - 1.1. [Goroutine Lifecycle](#11-goroutine-lifecycle)
   - 1.2. [Program Termination](#12-program-termination)
   - 1.3. [Summary](#13-summary)

# I. Goroutines

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

## 1.1. Goroutine Lifecycle

1. Created - goroutine is spawned
2. Running - actively executing
3. Sleeping - time.Sleep() or waiting
4. Terminated - function returns

## 1.2. Program Termination

Critical Point: When main() returns, the entire program exits, even if other goroutines are still running.

## 1.3. Summary

- Goroutines are lightweight and cheap
- Use go keyword to launch them
- They run concurrently, not in sequence
- Program exits when main() returns
- Perfect for I/O-bound tasks
