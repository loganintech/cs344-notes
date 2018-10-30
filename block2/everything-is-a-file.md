* Dirs are text files organizing hard links hierarchically
* Create or remove with these:
```c
mkdir
rmdir
```
* Because dirs are file, you can also read contents in C
```c
opendir
closedir
readdir
rewinddir
```
* Dir hard link represents itself (the `.`)
* Dir hard link representing parent (the `..`)

## What is a File in UNIX?

* A stream of bytes
* Could be accessed as an array
* Newlines / carriage returns & tabs are just bytes, too
* Persistent

## How do we access for reading and writing?

* Files can be opened as
```c
O_RDONLY // Readonly
O_WRONLY // Writeonly
O_RDRW // Read/Write
```
* When opening consider:
  * Should you delete an existing file with the same name?
  * If not, where do we start writing? Reading?
  * If it doesn't exist, should we create it?
  * If we create it, what should the permissions be?

## The File Pointer

* Tracks where the next file operation occurs in an open file
* Separate file pointer is maintained for each open file
* All of the operations we're talking about:
  * Directly impact which byte is pointed to by the file pointer when the file is opened
  * Move the file pointer
```c
fprintf // prints to a file
```

## Truncating
* To delete and start fresh, pass `O_TRUNC` in open
```c
open(filename, O_WRONLY | O_TRUNC)
```

## Appending
* To append to a file, pass O_APPEND
```c
open(filename, O_WRONLY | O_APPEND)
```

## Creating

* Open for writing only, creating it if it doesn't exist
```c
open(filename, O_WRONLY | O_CREAT, 0600) //0600 is the permissions
```
* Third param of open _must_ be used when creating a new file (ie using O_CREAT or O_TMPFILE)
* Access Permissions
  * Third param must be used when creating
  * Contains octal number permissions bits:
  * Specified as with `CHMOD`
  * Or can use `S_IRUSR | S_IWUSR`

## Lseek

* Manipulates a file pointer in a file
* Used to control where you're messing with da bitz `[sic]`

Examples:
  * Move to byte #16
```c
newpos = lseek(file_descriptor, 16, SEEK_SET);
```
* Move forward 4
```c
lseek(file_descriptor, 4, SEEK_CUR);
```
* Move back from end
```c
lseek(file_descriptor, 4, SEEK_END);
```

## Read()
* Reading AND Writing change the file pointer

## Standard IO in C

* Includes
```c
fopen
fclose
printf
fprintf
sprintf
scanf
fscanf
getc
putc
gets
fgets
fseek
```
* Makes it easy to work in line mode
* Why teach read() and write()?
  * Maximum performance
  * IF you know exactly what you are doing
  * No additional hidden overhead from stdio, which is much slower!
  * No hidden system calls behind stdio functions which may not be non-reentrant

## Files or Streams

* Stdin, stdout, and stderr are file streams, not file system files
* File streams wrap around, and provide buffering to, the underlying file descriptor among other features
* STDIO library streams are connected with fopen() to a variable of type FILE*
```c
FILE* myFile = fopen("foobarfilename", "r");
```
* Streams are closed when process terminates
* Raw file descriptors with open files are passed on to child processes
  * Process spawns a new child process with fork(), all open files are shared between parent and child processes
