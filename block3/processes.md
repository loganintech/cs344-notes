Video Source: https://www.youtube.com/watch?v=1R9h-H2UnLs


# Proper `waitpid()` placement

```c
pid_t spawnPid;
int childExitMethod;

spawnPid = fork();

if (spawnPid == -1) {
    perror("stuff");
    exit(1);
}
else if (spawnPid == 0) {
    waitpid(spawnPid);
    exit(0);
}
```

## Checking the Exit Status - Normal Termination

* `wait(&childExitMethod)` and `waitpid(..., &childExitMethod, ...)`
  * can identify two ways a process can terminate
* If the process terminates normally, then the WIFEXITED macro returns non-zero:
```c
if (WIFEXITED(childExitMethod) != 0)
  printf("The process exited normally\n")
```
* Get the actual exit status with WEXITSTATUS macro:
```c
int exitStatus = WEXITSTATUS(childExitMethod);
```
_Note:_ WEXITSTATUS always returns a value, so if the previous process exited it could return that. IE: If a process is killed by a signal it won't check to see if the signal ended it first and could get garbage or unrelated exit statuses.

## Checking the Exit Status - Signal Termination

_Teacher said to look it up if needed_

## Checking the Exit Status - Exclusivity
* Barring the use of non-standard `WCONTINUED` and `WUNTRACED` flags in `waitpid()`, only **one** of the `WIFEXITED()` and `WIFSIGNALED()` macros will be non-zero
* Thus, if you want to know how a child process died, you need to use both `WIFEXITED` and `WIFSIGNALED`
* A process that exits normally has no signal.
* A process that exits with a signal has no exit status.

## Checking the Exit Status - Code

```c
int childExitMethod;
pid_t childPID = wait(&childExitMethod);

if(childPID == -1) {
    perror("");
    exit(1);
}
```

## exec...() - Execute

* `exec...()` replaces the currently running program with a new program you specify
* The `exec...()` functions do not return - they destry the currently running program
  * No line after a successful `exec...()` call will run
* You can specify params by passing them to `exec`

## Two types of Execution

```c
int exel(char *path, char* arg1, ..., char* argn);
```
* executed program specified by `path` and gives it the command line arguments specified by strings `arg1` through `argn`
