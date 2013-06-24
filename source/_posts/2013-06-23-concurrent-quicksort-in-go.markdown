---
layout: post
title: "Concurrent quicksort in Go"
date: 2013-06-23 22:05
comments: true
categories: 
---
I've been playing around with Go to get some experience in a systems programming
language before applying for applying jobs. I've never done concurrent/parallel
programming so I thought I'd try to learn about some of Go's concurrency tools
by implementing quicksort concurrently.

I started by writing a simple in-place quicksort implementation. The majority of
the algorithm happens in a `partition` function which picks a random pivot and
then does in-place swapping to partition around the pivot. With this in place
the sequential algorithm falls into place easily.

```go
func Qsort(s []int) {
    if len(s) <= 1 {
        return
    }
    pivotIdx := partition(s)
    Qsort(s[:pivotIdx+1])
    Qsort(s[pivotIdx+1:])
    return
}
```

Since the divide and conquer nature of quicksort means the recursive calls to
Qsort don't touch the same memory, this should be a decent candidate for
concurrency.

```go
func Cqsort(s []int) {
    d := make(chan int)
    go cqsort(s, d)
    <-d
    return
}

func cqsort(s []int, done chan int) {
    // termination condition
    if len(s) <= 1 {
        done <- 1
        return
    }
    // where the work happens
    pivotIdx := partition(s)

    // communication channel
    childChan := make(chan int)
    go cqsort(s[:pivotIdx+1], childChan)
    go cqsort(s[pivotIdx+1:], childChan)

    // block until both concurrent children finish
    for i := 0; i < 2; i++ {
        <-childChan
    }
    // tell our caller we are done
    done <- 1
    return
}
```

This works but is incredibly slow. Goroutines are cheap but not free. For large
s this program spends most of its time in garbage collection as too many
goroutines are created.

We need to limit the number of goroutines.

```go
func Cqsort(s []int) {
	if len(s) <= 1 {
		return
	}
	workers := make(chan int, MAXGOROUTINES - 1)
	for i := 0; i < (MAXGOROUTINES-1); i++ {
		workers <- 1
	}
	cqsort(s, nil, workers)
}

func cqsort(s []int, done chan int, workers chan int) {
	// report to caller that we're finished
	if done != nil {
		defer func() { done <- 1 }()
	}

	if len(s) <= 1 {
		return
	}
	// since we may use the doneChannel synchronously
	// we need to buffer it so the synchronous code will
	// continue executing and not block waiting for a read
	doneChannel := make(chan int, 1)

	pivotIdx := partition(s)

	select {
	case <-workers:
		// if we have spare workers, use a goroutine
		// for parallelization
		go cqsort(s[:pivotIdx+1], doneChannel, workers)
	default:
		// if no spare workers, sort synchronously
		cqsort(s[:pivotIdx+1], nil, workers)
		// calling this here as opposed to using the defer
		doneChannel <- 1
	}
	// use the existing goroutine to sort above the pivot
	cqsort(s[pivotIdx+1:], nil, workers)
	// if we used a goroutine we'll need to wait for
	// the async signal on this channel, if not there
	// will already be a value in the channel and it shouldn't block
	<-doneChannel
	return
}
```

Setting MAXGOROUTINES limits the number of goroutines that ever get created by
switching to a synchronous model once the workers channel is exhausted.

This performs much better than before. Benchmarking puts it at about the same
speed as the built-in sort.Ints, but still slower than our basic Qsort
implementation for a shuffled slice of 10 million elements created by
math/rand.Perm. Playing around with MAXGOROUTINES I seemed to get slight
variations when using numbers anywhere from 2 to 10000 (make sure you have the
GOMAXPROCS environment variable set to the number of cores you want to use
so that goroutines are executed in parallel). Above 10000 and performance starts
to degrade.

This use of Go's concurrency features purely for the purpose of parallelizing
code goes against Rob Pike's "Concurrency is not Parallelism" talk, so I'm not
surprised that I don't see a significant speedup when trying to parallelize this
concurrent code against the sequential version. I am surprised that I don't get
similar speeds to the sequential version when setting MAXGOROUTINES to 1 as the
code is essentially executing sequentially in this case. Perhaps the few extra
statements and branches are enough to slow things down considerably. In any case
it was a good experiment in writing concurrent Go that also forced me to dive
into profiling with pprof and using Go's support for writing benchmarks in test
code.
