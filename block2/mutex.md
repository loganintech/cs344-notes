## Mutexes
* Abbreviation for Mutual Exclusion
* Implemented as part of the POSIX pthread API to provide thread syncronization via "locks"
* Provides the programmer the ability to protect data from multiple reads and writes on the same files
* There are several types which check variously for errors, perform faster, etc.
## Lifespan
* Athread acquires a lock on a mutex variable by attempting to call a special lock function
* Pthread_mutex_t myMutex = PTHREAD_MUTEX_INITIALIZER
* Pthread_mutex_destroy(myMutex)
* Once the mutex is locked, any other thread attempting to lock it will block

## Unlocking

* A thread unlocks a mutex variable that it has previously locked like so:
  * `pthread_mutex_unlock(myMutex);`



## Pseudo Code
```
Main() thread            - Time thread
Lock mutex
Create () time thread
… print time
Loop() {
    if ("time") {
        unlock mutex
        join() time thread
        lock mutex
        create() time thread
     }
}
```
