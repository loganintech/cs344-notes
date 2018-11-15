# Network Clients

## Berkeley Sockets
* Dev'd in early 1980s for BSD Unix under DARPA grant
* The _de facto_ network communication method across local area networks and the internet.
  * Other transmission methods exist, but require different transport protocols
    * You'd have to write your own TCP if you want to use something other than sockets.

## Ports
* Example: A house has address 32. Each open window would be an open port.
* _Ports_ are used to reach a specific process
* Each process listens on a unique port - similar to unique entrance to the house
* So a complete address can be used in a socket is an IP address combined with port number.

## Socket Documentation
* Most socket related an pages are in the "3n" section

```sh
man -s 3n socket
man -k socket
```

* All the info you need to use the netowrk library is scattered across different man pages
* It's definitely best to work from a known good network program starting point

## Creating and Connecting a Socket on the Client

* Process:
  * Create the socket endpoint with `socket()`
  * Conect the socket to the server with `connect()`
  * Use `read()` and `write()`, or `send()` and `recv()`, to transfer data to and from the socket (which is sent automatically to and from socket on server)

* Sockets act like files, in that you can `read()` and `write()` to them
* `send()` and `recv()` are specialized and can take special flags

## Creating the Socket

```c
int socket(int domain, int type, int protocol);
```

* Returns file descriptor or -1
* Domain for general purpose sockets can connect across a network, use `AF_INET`
  * For sockets that are used ONLY for same-machine IPC, use `AF_UNIX`
* Type is SOCK_STREAM or SOCK_DGRAM
* Protocol is pretty much always 0

## Connecting the Socket to an Address

```c
int connect(int sockfd, struct sockaddr* address, size_t address_size);
```

* Returns 0 on success, -1 on failure
* Socketfd is socket you want to connect to
* `sockaddr` is a struct that holds the address of where you're connecting, plus other settings
* Pass address_struct's size to the last parameter

## Filling the Address Struct

### IP Address

* Getting the actual address into a form `connect()` can use is tricky:

```c
struct sockaddr_in serverAddress;

serverAddress.sin_family = AF_INET;
serverAddress.sin_port = htons(7000);
serverAddress.sin_addr.s_addr = inet_addr("192.168.1.1");
```

### Domain Name

```c
struct sockaddr_in serverAddress;
struct hostent* serverHostInfo;

serverHostInfo = gethostbyname("www.oregonstate.edu");
if (serverHostInfo == NULL) {
    fprintf(stderr, "Couldn't resolve hostname.\n");
    exit(1);
}

serverAddress.sin_family = AF_INET;
serverAddress.sin_port = htons(80);

memcpy((char*)&serverAddress.sin_addr.s_addr, (char*)serverHostInfo->h_addr, serverHostInfo->h_length);
```

## Sending Data
* Send will block until all data has been sent, the connection goes away, or a signal handler interrupts the `write()` system call.
* Remember that internet connections fail all the time
  * Client intentionally disconnects (STOP button in a web browser)
  * Network failure
  * etc.

```c
ssize_t send(int sockfd, void *message, size_t message_size, int flags);

char msg[1024];

r = send(socketFD, msg, 1024, 0);
if (r < 1024)
    {} // handle possible error
```

## Receiving Data

```c
ssize_t recv(int sockfd, void *buffer, size_t buffer_size, int flags);

char buffer[1024];
memset(buffer, '\0', sizeof(buffer));
r = recv(socketFD, buffer, sizeof(buffer) - 1, 0);
if (r < sizeof(buffer) - 1)
    {} // handle possible error
```

* Data may arrive in odd size bundles!
* `recv()` or `read()` will return exactly the amount of data that has already arrived
* `recv()` and `read()` will block if the connection is open but no data is available
  * So be careful to match what you send with what you receive, or use:
    ```c
    fcntl(socketFD, F_SETFL, O_NONBLOCK);
    ```
  * Sets the socket to block if there is no data, but that means you're polling the socket, waiting for data; `select()` would be better (see next lecture!)

### Using Control Codes

* Similar to controlling data being sent through pipes, you can watch for the amount of data coming through `recv()` if you know how much there should be, or use codes:

```c
char completeMessage[512], readBuffer[10];
memset(completeMessage, '\0', sizeof(completeMessage)); // Clear the buffer
while (strstr(completeMessage, "@@") == NULL) // As long as we haven't found the terminal...
{
    memset(readBuffer, '\0', sizeof(readBuffer)); // Clear the buffer
    r = recv(socketFD, readBuffer, sizeof(readBuffer) - 1, 0); // Get the next chunk
    strcat(completeMessage, readBuffer); // Add that chunk to what we have so far
    printf("PARENT: Message received from child: \"%s\", total: \"%s\"\n", readBuffer, completeMessage);
    if (r == -1) { printf("r == -1\n"); break; } // Check for errors
    if (r == 0) { printf("r == 0\n"); break; }
}

int terminalLocation = strstr(completeMessage, "@@") - completeMessage; // Where is the terminal
completeMessage[terminalLocation] = '\0'; // End the string early to wipe out the terminal
printf("PARENT: Complete string: \"%s\"\n", completeMessage);
```
