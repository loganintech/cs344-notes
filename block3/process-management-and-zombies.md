# Process Management and Zombies

## Running Processes
* How can we tell which processes are running? Use the `ps` command to get information about running processes.
* `ps` by itself is really boring, and not useful
```sh
$ ps
PID     TTY     TIME        CMD
18779   pts/8   00:00:00    bash
18934   pts/8   00:00:00    ps
```

## Brewster's Suggestion

```sh
alias psme='ps -o ppid,pid,euser,stat,%cpu,rss,args | head -n 1; ps -eH -o
ppid,pid,euser,stat,%cpu,rss,args | grep brewsteb'
```

Results in

```sh
$ psme
PPID    PID     EUSER       STAT    %CPU    RSS     COMMAND
4533    18776   root        Ss 0.2  4284    sshd:   brewsteb [priv]
18776   18778   brewsteb    S 0.0   2112    sshd:   brewsteb@pts/8
18778   18779   brewsteb    Ss      0.0     2044    -bash
18779   18911   brewsteb    R+      4.0     1840    ps   -eH    -o  ppid,pid,euser,stat,%cpu,rss,args
18779   18912  brewsteb     S+      0.0     820     grep  brewsteb
```

* PPID - Parent Process ID
* PID - Process ID
* EUSER - Effective User ID
* STAT - Execution State
* %CPU - Percentage of CPU time thisprocess occupies
* RSS Real Set Size - kilobytes of RAM in use by this process
* Command - The actual command the user entered

### First State Character

* D Uninterruptible sleep (usually IO)
* R Running or runnable (on run queue)
* S Interruptible sleep (waiting for an event to complete)
* T Stopped, either by a job control signal or because it is being traced
* Z Defunct ("zombie") process, terminated but not reaped by its parent

### Second State Character (Optional)

* < High-priority (not nice to other users)
* N Low-priority (nice to other users)
* L Has pages locked into memory (for real-time and custom IO)
* s Is a session leader (closes all child processes on termination)
* L Is multi-threaded (Uses pthread)
* + Is in the foreground process group


## Zombie

* When a child process terminates, but its parent does not wait for it, the process becomes a zombie.
* Child processes must report to their parents before their resources will be released by the OS
* If the parents aren't waiting for their children, the process becomes the _living undead - forever consuming, forever enslaved to a non-life of waiting and watching._
* The purpose of a zombie process is to retain the state that `wait()` can retrieve; they _want_ to be harvested

## Makin Zombies
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
void main()
{
    pid_t spawnPid = -5;
    int childExitStatus = -5;
    spawnPid = fork(); //returns negative on failure, 0 to the new child, and positive for the new process id
    switch (spawnPid)
    {
    case -1:
        perror("Hull Breach!\n");
        exit(1);
        break;
    case 0:
        printf("CHILD: Terminating!\n");
        break;
    default:
        printf("PARENT: making child a zombie for ten seconds;\n");
        printf("PARENT: Type \"ps -elf | grep \'username\'\" to see the defunct child\n");
        printf("PARENT: Sleeping...\n");
        fflush(stdout); //Flush stdout before sleeping so everything is printed.
        sleep(10);
        waitpid(spawnPid, &childExitStatus, 0);
        break;
    }
    printf("This will be executed by both of us!\n");
    exit(0);
}
```

## Orphan Zombies
* If a parent process terminates without cleaning up its zombies, the zombieis become orphan zombies
* Oprhans are adopted by the `init` process (usually pid = 1) which pediodically (in practice, very quickly) `wait()`s for orpahns
* So, orphans die

## Kill
* The `UNIX` command used to kill programs
  * old version: `kfork`
* `kill` is a misnomer, it _just sends signals_

```sh
kill -TERM $foopid
```
* Given PID affects who the signal is sent to
  * if PID > 0, signal is sent to PID given
  * if PID == 0, signal is sent to all processes in the same process group as the sender, ie the shell
  * More trickiness for PID < 0
* Signals will be discussed in later lectures more in-depth

## Top
* `top` allows you to view the process running on the machine in real time - one of the few animated built-in programs

## Diagnosing a Slow CPU
* The _uptime_ command shows the average number of runnable processes over several different periods of time (same info top displays)

```sh
$ uptime
1:23pm up 25 day(s), 5:59, 72 users, load average: 0.18, 0.19, 0.20
```
* This shows the average number of runnable (the current running process plus the queue of processes waiting to be run) or uninterruptable (waiting for IO) processes over the last 1, 5, and 15 minutes.
* If uptime is showing that the runnable queue is consistently _larger than the number of cores_, your CPU is a bottleneck and is causing slow-down

## Get CPU Info

```sh
$ cat /proc/cpuinfo
processor        : 0
vendor_id        : GenuineIntel
cpu family       : 6
model            : 45
model name       : Intel(R) Xeon(R) CPU E5-2665 0 @ 2.40GHz
stepping         : 7
microcode        : 1808
cpu MHz          : 2399.993
cache size       : 20480 KB
physical id      : 0
siblings         : 16
core id          : 0
cpu cores        : 8
apicid           : 0
initial apicid   : 0
fpu              : yes
fpu_exception    : yes
```

## Diagnosing a Slow CPU - Single Core
```sh
$ uptime
14:33:04 up 34 days, 5:34, 10 users, load average: 0.05, 0.15, 0.20 # CPU is fast or not doing anything
$ uptime
14:33:04 up 34 days, 5:34, 10 users, load average: 0.88, 1.03, 0.96 # CPU is around max time, look to upgrade soon
$ uptime
14:33:04 up 34 days, 5:34, 10 users, load average: 4.79, 7.23, 6.44 # Shit has officially hit the fan
```
*Note*: Multiply the above number by the number of cores. For 8 cores, you would want 8.0 for max time.


## Foreground / Background
* There can be only one shell **foreground** process - it's the one being interacted with
* In a command prompt, the foreground program is the shell itself
* Processes in the background can still be executing, but they can be in any number of stopped states. (See info from above about first state character)

## Foreground / Background (in reality)

* There really isn’t any difference between processes in these two states; its merely shell nomenclature used to distinguish between them
* When a user enters a command that is intended to run in the foreground (i.e. a normal command), the process started runs to completion before the
user is prompted again
* When a user enters a command that is intended to run in the background (see later slides), the user is immediately prompted again after the process is executed
* In other words, control input to the terminal is not interrupted by a background process

## Start Backgrounded
Just add `&` at the end of the command.
Ex:
```sh
$ ping www.oregonstate.edu &
```

* The ampersand means to start in the background, and must be the last character
* Note that stdout and stderr are still going to the terminal for that process, and stdin might be too if the shell is badly programmed

## Stopping a Process
* Sending the `TSTP` signal stops (not terminates) a process, and puts it into the background
  * Ctrl+Z also sends the signal

```sh
$ ping www.oregonstate.edu
PING www.orst.edu (128.193.4.112) 56(84) bytes of data.
64 bytes from www.orst.edu (128.193.4.112): icmp_seq=1 ttl=250 time=0.362 ms
64 bytes from www.orst.edu (128.193.4.112): icmp_seq=2 ttl=250 time=0.321 ms
64 bytes from www.orst.edu (128.193.4.112): icmp_seq=3 ttl=250 time=0.324 ms
64 bytes from www.orst.edu (128.193.4.112): icmp_seq=4 ttl=250 time=0.328 ms

^Z
[1]+ Stopped ping www.oregonstate.edu

$
```


## Jobs

* Use the jobs command to see what you’re running:

```sh
$ ping www.oregonstate.edu
PING www.orst.edu (128.193.4.112) 56(84) bytes of data.
64 bytes from www.orst.edu (128.193.4.112)
64 bytes from www.orst.edu (128.193.4.112)

^Z
[3]+ Stopped ping www.oregonstate.edu

$ jobs -l
[1] 31314 Stopped ping www.oregonstate.edu
[2]- 31317 Stopped ping www.oregonstate.edu
[3]+ 31327 Stopped ping www.oregonstate.edu

$ kill -KILL 31327
[3]+ Killed ping www.oregonstate.edu

$ kill -KILL %1
[1]- Killed ping www.oregonstate.edu

```
## Fg
* Use job numbers from `jobs` to manipulate

* Bring job 1 from the background to the foreground, and start it running again

```sh
fg %1
```

* Bring most recent backgrounded job to the foreground, and start it running again
```sh
fg
```

## Bg

* Start a specific stopped program that is currently in the background (and keep it in the background):
```sh
bg %1
```

* Start the most recently stopped program in the background (and keep it in the background):
```sh
bg
```

## Who's got control of STDOUT?
* Be advised - BG processes can still write to any file including `stdout` and `sterr`
Demo:

```sh
ping www.oregonstate.edu
CTRL-Z
jobs
fg %1
CTRL-Z
jobs
bg %1
ps
CTRL-Z CTRL-Z CTRL-Z (doesn’t do anything)
fg %1
CTRL-C
```

## You're suspended

* Suspend a process that is currently running in the background when you’re at the shell

```sh
# send stderr and stdout somewhere other than the terminal and background it
$ ping www.oregonstate.edu 2>/dev/null 1>logfile &
[1] 1660

$ jobs
[1]+ Running ping www.oregonstate.edu 2> /dev/null > /dev/null &

# Send the stop signal
$ kill -TSTP %1
[1]+ Stopped ping www.oregonstate.edu 2> /dev/null > /dev/null
$ jobs
[1]+ Stopped ping www.oregonstate.edu 2> /dev/null > /dev/null
```

## History

* The history command provides a listing of your previous commands
```sh
$ history 5
1012 jobs
1013 psme
1014 top
1015 jobs
1016 history 5
```

## Execute previous command
```sh
$ history 3
1030 jobs
1031 psme
1032 history 3

$ !1030
jobs

$ history 3
1032 history 3
1033 jobs
1034 history 3

$ !-2
jobs

$ !!
jobs

$ history 3
1034 history 3
1035 jobs
1036 history 3
```
