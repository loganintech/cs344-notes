* Def'n
  * Program decomposition into order-independent chunks
* On UNIX concurrency is easy
  * Multiple processes can be running simultaneously
  * Multiple copies of the same program can be running
  * CPU time can be split and shared
* Concurrency is very powerful
  * Greatly increases efficiency: While one process is waiting for I/O, another can use the CPU (or other parts)

## Definitions: System Calls vs Library Functions

* System Calls
  * Request for service that causes normal operation of a process to be interrupted and control passes to the OS
  * Typically, process is now blocked and won't do anything else until the system call returns
  * Ex: `read()`, `write()`, etc.
* C Library Functions
  * Faster, as they have no permissions or blocking issues
  * Ex: `sqrt()`, `printf()`, etc.

## Illusions of Simultaneous Execution

* Multiprogramming
  * More than one process can be ready to execute
  * System calls trigger "context switches", which let the next process run
  * The process will not execute again until tits system call returns
* Timesharing
  * CPU time split between multiple processes
  * Gives illusion that many processes are running at once

## Multiprocessing

* Def'n: Executing multiple processes at the actual same time
* Today's CPUs have 4, 6, 8, and 10 cores, with more coming
  * Each core acts like a mini-CPU, each of which can do multiprogramming AND timesharing

## Possible Complications

* Concurrently running processes can share data and/or resources
* What is multiple processes access the same resource at the same time?
  * This often results in a datarace

## Race conditions

* Why are they hard to detect?
  * They may only show up once in a strange or unreproducible set of conditions

## Seriousness of Software Engineering

* Most infamous race condition
  * https://en.wikipedia.org/wiki/Therac_25
  * People could outrun the system
    * The system was counting on a normally slow human, and didn't take into account people learning to use the system faster
  * 3 people died, 3 were injured as a result of this mistake

## The Lesson - Access Control
* Concurrent update situation
  * 2+ Processes accessing resource concurrently
  * At least one process might write
* Must provide access control
  * If one process is writing, no other process should access (read OR write) the resource
* "Locks" solve these problems
  * Only process owning the lock may access

## Deeper Definitions

* Kernel
  * The central part of the OS
  * Manages hardware (and drivers)
  * Provides the Scheduler
  * Not interacted with by users: system calls are requests to the kernel

* Scheduler
  * Distributes prioritized and/or fair CPU time

## Thread Advantages over Processes

* Communication between threads is vastly simpler than IPC:
  * Threads share the following:
    * Code, Heap, Data
  * They each have their own Stack, but can access the Stacks of other threads
* The CPU switches between executing threads much faster than switching between processes, because of what's shared above
  * On a multi-core CPU, the CPU can run each kernel-level thread on a separate core, yielding true concurrency



## Thread Disadvantages

* Shared resources means increased possibilities of:
  * Race conditions
  * Resource contention, including file access
  * Leaking of sensitive data across threads

## Types of Threads

* Kernel Level Threads
  * Controlled by the kernel, given time by the scheduler
  * Akin to a mini-process
  * Simple to create with built-in libraries in UNIX
  * Generally considered the better choice in almost all circumstances
* User Level Threads
  * Kernel doesn't know about these - entirely within a process and do not involve the scheduler; Switching between them is done cooperatively by the protocol of the software itself to manage the task
  * Real an emulation of threading
  * Sometimes called green threads
  * Switching between user-level threads is faster than switching between kernel-level threads, because it doesn't require swapping memory protection

## Thread Implementation in UNIX

* Implemented with the POSIX Threads API in UNIX
  * Compile with -lpthread option in gcc
```c
#inclue <pthread.h>
```

* Creating
```c
int pthread_create( pthread_t* thread, const pthread_attr_t* attr, void* (*start_routine) (void *), void* arg);
```

* Thread points to the variable in which the ID of the new thread is written into; depending on OS implementation, sometimes this is an int, sometimes it's a struct
* Attr points to constant thread flags
* `*start_routine` points to a function (in the current program) that will be the start point of executionfor the new thread that copies this one
  * `(void *)` is a function pointer
* Arg points to the sole argument that is passed into `start_routine(NULL if none)`; if multiple arguments are desired, pass a struct

Example
```c
int resultInt;
pthread_t myThreadID;
resultInt = pthread_create(&myThreadId, NULL, start_routine, NULL);
```
* Note there is no () after start_routine because function pointer doesn't take

* Killing a Thread
  * Thread calls pthread_exit()
  * Thread returns from start_routine()
  * Thread cancelled by another thread with pthread_cancel()
  * Any thread in the process calls exit()
* Identifying a Thread
```c
pthread_t myThreadID = pthread_self();
pthread_equal(myThreadID, unknownThreadID);
```
