---
layout: post
title: "Concurrent Qsort in Go - Part II"
date: 2013-06-25 13:46
comments: true
categories: [go, concurrency]
---

After my [past attempt](https://azundo.github.io/blog/concurrent-quicksort-in-go) at writing a
concurrent Qsort algorithm I had a few more thoughts about how to change the
architecture.

Instead of creating new channels to pass to every recursively-called goroutine
and then blocking on them until they return, I wanted to create only a single
channel for communication and find a way to return from goroutines immediately.
The solution I came up with is to send back the integer value of the number of
elements in their sorted position when a goroutine returns. In the naive case,
this is when the list is 0 or 1 elements long, leaving this fairly simple code.

```go

func Cqsort2(s []int) {
    // this channel will collect the number of sorted elements
	returnChannel := make(chan int)
	go cqsort2(s, returnChannel)
    // sum up the number of sorted elements until they equal
    // the length of our original slice
	for sortedElements := 0; sortedElements < len(s); {
		sortedElements += <-returnChannel
	}
	return
}

func cqsort2(s []int, done chan int) {
	if len(s) <= 1 {
	    // report the number of sorted elements to the
        // communication channel
		done <-len(s)
		return
	}

	pivotIdx := partition(s)

	go cqsort2(s[:pivotIdx+1], done)
	go cqsort2(s[pivotIdx+1:], done)
    // this goroutine should now return immediately and
    // doesn't have to block on the child calls
	return
}
```

This still creates too many goroutines and uses too much memory. So taking a
page from most quicksort implementations, lets use a selection sort for the last
few elements. Empirically I found a value of about 75 to be ok. Our overall
architecture doesn't have to change, just add the selection sort call in the
termination condition of our qsort2 function.

```go

func cqsort2(s []int, done chan int) {
	if len(s) <= 75 {
        // run a selection sort when s is small
        selectionSort(s)
	    // report the number of sorted elements to the
        // communication channel
		done <-len(s)
		return
	}

	pivotIdx := partition(s)

	go cqsort2(s[:pivotIdx+1], done)
	go cqsort2(s[pivotIdx+1:], done)
    // this goroutine should now return immediately and
    // doesn't have to block on the child calls
	return
}
```

Finally when using GOMAXPROCS=2 I got a better running time for the concurrent
version over the non-concurrent Qsort from my last post. This isn't quite a fair
comparison though because I wasn't using the selectionSort trick in Qsort.
Adding it in to Qsort and our previous Cqsort brings all three implementations
to about 7s to sort 1 milllion elements. So still no improvements over a
sequential implementation despite using both of my cores at 100% in the
concurrent cases. Memory and communication overhead still provide too much of a
slow down.  I'm hoping to get my hands on a computer with 4 cores to see if the
concurrent version will see a speedup when doubling the core count again, or if
communication overhead is still too high.

Again, all of the code and some benchmarking is in a github repository at
[https://github.com/azundo/cqsort.git](https://github.com/azundo/cqsort.git).
