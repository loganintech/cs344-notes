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
    perror("wait failed");
    exit(1);
}

if (WIFEXITED(childExitMethod)) {
    printf("The process exited normally\n");
    int exitStatus = WEXITSTATUS(childExitMethod);
    printf("exit status was %d\n", exitStatus);
} else {
    printf("Child terminated by a signal\n");
}
```

## exec...() - Execute

* `exec...()` replaces the currently running program with a new program you specify
* The `exec...()` functions do not return - they destry the currently running program
  * No line after a successful `exec...()` call will run
* You can specify params by passing them to `exec`

## Two types of Execution

```c
int execl(char *path, char* arg1, ..., char* argn);
```
* executed program specified by `path` and gives it the command line arguments specified by strings `arg1` through `argn`


```c
int execv(char *path, char *argv[]);
```
* Executes the program specified by path, and gives it the command
line arguments indicated by the pointers in argv

## Current Working Directory

* `execl()` and `execv()` do not examine the PATH variable - they
only look in the current working directory (but see the next slide)
* if you don't specify a fully-qualified name, programs will not be executed, even if they are in a dir listed in PATH, and `exelc()` and `execv()` will return error
* Change it with `getcwd()` and `chdir()`

## Exec...() and the PATH variable

```c
int execl(char *path, char *arg1, ..., char *argn);
int execlp(char *path, char *arg1, ..., char *argn);
int execv(char *path, char *argv[]);
int execvp(char *path, char *argv[]);
```

* The versions ending in `P` will search through the `PATH` variable

## Execute a New Process

* `exec...()` _replaces_ the program it is called from - it does not create a new process.
* use `fork()` and `exec...()`, we can keep our original program going, and spawn a brand-new process

## Passing parameters to `execlp()`

```c
int execlp(char *path, char *arg1, ..., char *argn);
```
* First parameter to `execlp()` is the pathname of the new program
* remaining parameters are "command line arguments"
* First argument should always be the same as the first parameter
* Last argument must always be NULL, which indicates that there are no more parameters
* Do not pass any shell-specific operators into any member of the `exec...()` family, like `<,>,|,&,!`, because the shell is not being invoked, only the OS

Example:
```c
execlp("ls", "ls", "-a", NULL);
```

## `fork()` + `execlp()` Example

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
void main() {
    pid_t spawnPid = -5;
    int childExitStatus = -5;
    spawnPid = fork();

    switch (spawnPid) {
        case -1: { perror("Hull Breach!\n"); exit(1); break; }
        case 0: {
            printf("CHILD(%d): Sleeping for 1 second\n", getpid());
            sleep(1);
            printf("CHILD(%d): Converting into \'ls -a\'\n", getpid());
            execlp("ls", "ls", "-a", NULL);
            perror("CHILD: exec failure!\n");
            exit(2);
            break;
        }
        default: {
            printf("PARENT(%d): Sleeping for 2 seconds\n", getpid());
            sleep(2);
            printf("PARENT(%d): Wait()ing for child(%d) to terminate\n", getpid(), spawnPid);
            pid_t actualPid = waitpid(spawnPid, &childExitStatus, 0);
            printf("PARENT(%d): Child(%d) terminated, Exiting!\n", getpid(), actualPid);
            exit(0); break;
        }
    }
}
```

## Passing paramteres to `execvp()`

```c
int execvp(char *path, char *argv[])
```
* First parameter to execvp() is the pathname of the new program
* Second parameter is an array of pointers to strings
* First string should be the same as the first parameter (the command itself)
* Last string must always be NULL, which indicates that there are no more parameters
* • Do not pass any shell-specific operators into any member of the exec…() family, like <,
>, |, &, or !, because the shell is not being invoked - only the OS is!

## Full `execvp()` example

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

void execute(char** argv) {
    if (execvp(*argv, argv) < 0) {
        perror("Exec failure!");
        exit(1);
    }
}

void main() {
    char* args[3] = {"ls", "-a", NULL};
    printf("Replacing process with: %s %s\n", args[0], args[1]);
    execute(args);
}
```

## `exit()`

* `atexit()`
    * Arranges for a function to be called before exit()
* `exit()` does the following:
    * Calls all functions registered by atexit()
    * Flushes all stdio output streams
    * Removes files created by tmpfile()
    * Then calls _exit()
* `_exit()` does the following:
    * Closes all files
    * Cleans up everything - see the man page for `wait()` for a complete list of what happens on exit
* `return()` from `main()` does exactly the same thing as `exit()`

## Environment Variables

* A set of text variables, often used to pass information between the
shell and a C program
* May be useful if:
    * You need to specify a configuration for a program that you call frequently
(LESS, MORE)
    * You need to specify a configuration that will affect many different commands
that you execute (TERM, PAGER, PRINTER)
* You can view/edit the environment from bash by using the
printenv and export commands, and assignment (=) operator
* The environment can be edited in C with setenv() and getenv()

## Manipulating the Environment

* Bash:
```bash
MYVAR="Some text"
export MYVAR
echo $MYVAR
MYVAR="New text"
```
* C:
```c
setenv("MYVAR", "Some text string 1234", 1);
printf("%s\n", getenv("MYVAR"));
```
## Manipulating the Environment… for Just You

```c
$ cat bashAndCEnvironment.c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
int main(int argc, char* argv[])
{
    char array[1000];
    printf("Variable %s has value: %s\n", argv[1], getenv(argv[1]));
    printf("Doubling it!\n");
    strcpy(array, getenv(argv[1]));
    strcat(array, getenv(argv[1]));
    printf("New value of %s will be: %s\n", argv[1], array);
    setenv(argv[1], array, 1);
    printf("Variable %s has value: %s\n", argv[1], getenv(argv[1]));
}
```

```sh
$ MYVAR="TEXT."
$ export MYVAR
$ echo $MYVAR
TEXT.
$ gcc -g -o bashAndCEnvironment bashAndCEnvironment.c
$ bashAndCEnvironment MYVAR
Variable MYVAR has value: TEXT.
Doubling it!
New value of MYVAR will be: TEXT.TEXT.
Variable MYVAR has value: TEXT.TEXT.
$ echo MYVAR
TEXT.
```

## Exporting Environment Variables
```sh
$ MYTESTVAR="testtext"
$ echo $MYTESTVAR
testtext
$ bashAndCEnvironment MYTESTVAR
Variable MYTESTVAR has value: (null)
Doubling it!
Segmentation fault (core dumped)
$ export MYTESTVAR
$ bashAndCEnvironment MYTESTVAR
Variable MYTESTVAR has value: testtext
Doubling it!
New value of MYTESTVAR will be: testtexttesttext
Variable MYTESTVAR has value: testtexttesttext
$ echo $MYTESTVAR
testtext
```

## Fork Bombs
* Yes, they’re hilarious
* Under no circumstances should you be running systems development code
on any non-OS class server!
* Consider the following warning signs that you might be about to do
something dangerous, where if something goes wrong, your program
might consume all of the system resources available and lock you and
everyone else out:
    * You've written a loop that calls fork()
    * You've written code in which your child process creates another child process (a
fork() within a forked process; these are usually not what you want)
    * You've written code in which your child process is starting up a loop

* Remember that you need to be really extra
sure that you have termination methods
built-in to your loops
* Consider having a variable set a flag called
forkNow in your loop. Then, have a
separate function call fork() because the
flag value was set, with this function also
resetting the flag value at the end
* Consider during testing, for example, adding
an extra condition to a loop with a counting
variable: if you hit 50 forks, say, then
abort()
