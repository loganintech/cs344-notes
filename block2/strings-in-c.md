# Strings in C don't exist

There _is_ a list of characters, ended by a null value "\0"

* Formatted means numbers correctly printed with text
* Formatted printing is done with
```c
printf // write to stdout
sprintf // write to other char*
fprintf // write to file
```
Basic C String Funcs
```c
strcmp
strlen
strcpy
strcat
```
N-char versions
```c
strncpy
strncat
```

## Clean array values
```c
memset(array, whattoset, length);
```
Ex
```c
char mystring[20];
memset(mystring, '\0', sizeof(mystring));
```
## Back in dangerous C
```c
strtok()
```
* Splits strings into chunks
* Makes hair fall out
* Maxes out credit cards
* Unfriends all your social media friends
* Sometimes the only tool for the job

Example
```c
char input[18] = "This.is my/string";
char* token = strtok(input, " ./"); //This
token = strtok(NULL, " ./"); //is
token = strtok(NULL, " "); //my/string
```

* First drawback, of many
  * It edits the input string with null terminators, so if you give it static data it segfaults
* Second
  * After the first string to tokenize, you just pass NULL from then on. You can't mix calls of string tokenizer
  * strtok_r achieves re-entrancy, allowing the mixing of calls, by requiring you pass in a pointer to a temp variable for it to use
