---
layout: default
title: "Lecture 15: Socket programming in C"
---

# Unit File I/O, Sockets

Demo programs:

> [socket.zip](socket.zip)

## TCP/IP Socket programming

Sockets &mdash; the Unix standard API for network I/O

> Windows has something almost identical, called "winsock"

## File descriptors

A file descriptor is an integer value naming an underlying open file resource.

File descriptors are used in all Unix system calls which do I/O, or other operations on files/streams.

### read and write

`read` &mdash; read data from a file/stream named by a file descriptor into a buffer

{% highlight cpp %}
ssize_t read(int fd, void *buf, size_t count);
{% endhighlight %}

`write` &mdash; write data from a buffer to a file/stream named by a file descriptor

{% highlight cpp %}
ssize_t write(int fd, const void *buf, size_t count);
{% endhighlight %}

See the `write_to_file.c` demo program.

### fdopen

`read` and `write` are a pain. For example, no formatted input/output.

Solution: convert a file descriptor into a FILE \* using the fdopen() function.  Then, use standard I/O functions such as `fprintf`, `fgets`, and `fscanf` to do I/O

Note that attempting to use a single FILE\* to read from and write from the pipe will not necessarily work.  (TODO: figure out why!)  A work-around is to use the `dup` system call to duplicate the socket file descriptor, and then make two calls to `fdopen` to create file handles for reading an writing:

{% highlight c %}
int fd = socket(...);

FILE *read_fh = fdopen(fd, "r");
FILE *write_fh = fdopen(fd, "w");
{% endhighlight %}

## Unix man pages

The Unix `man` command shows Unix manual pages.  These are extremely useful for getting quick reference documentation on system calls and C library functions.

Unix system calls (such as `read` and `write`) are documented in section 2 of the manual, and standard C library functions (such as `fdopen`) are documented in section 3 of the manual.

Some example commands to try:

    man 2 open
    man 2 read
    man 2 write
    man 3 fdopen

## Sockets

A socket is a *communications endpoint*.

Datagram socket: Can send/receive *datagrams* &mdash; discrete chunks of data. Analogy: mailbox.

Stream socket: is one end of a *connection*. A connection is a private communication channel between two communicating processes. Analogy: telephone.

Important concepts:

> Address &mdash; uniquely identifies a *host* on a network
>
> Port &mdash; distinguishes sockets on the same host from each other.

At the time the socket API was created (at UC Berkeley in the early 1980s), a variety of network protocol stacks were in active use, such as

-   TCP/IP
-   DECNET
-   others?

For this reason, the socket API treats socket addresses as an opaque data type, with multiple address types as "subclasses".

`sockaddr\_un` &mdash; a union of all available socket types

## Socket API

The socket API is implemented in a number of system calls.

`socket` &mdash; create a socket

{% highlight cpp %}
int socket(int domain, int type, int protocol);
{% endhighlight %}

* domain &mdash; what protocol family the socket will use. PF\_INET for TCP/IP
* type &mdash; SOCK\_STREAM for stream, SOCK\_DGRAM for datagram
* protocol &mdash; identifies a particular protocol to use, if not uniquely determined by type. Just use 0 for any TCP/IP socket.

The `socket` system call is typically used to create a *server socket*, which allows a process to accept incoming connections.

`listen` &mdash; wait for incoming connections

{% highlight cpp %}
int listen(int sockfd, int backlog);
{% endhighlight %}

* sockfd &mdash; the server socket on which to listen for incoming connections
* backlog &mdash; maximum number of incoming connections which will be queued

The return value is 0 if a connection has been received, or -1 if an error occurred.

`accept` &mdash; accept an incoming connection

{% highlight cpp %}
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
{% endhighlight %}

* sockfd &mdash; the server socket file descriptor
* addr &mdash; a pointer to a `sockaddr` struct (really, a "subclass" of `sockaddr`) where the client's network address will be stored
* addrlen &ndash; the size in bytes of the struct that addr points to

The return value is a file descriptor naming a socket connected to a two-way communications channel to the client.

## Example programs

* write\_to\_file.c &mdash; open a file and write to it using Unix system calls
* server.c &mdash; a simple server program implemented using Unix system calls for I/O
* server2.c &mdash; similar to server.c, but using `fdopen` and standard I/O functions
* client.c &mdash; a simple client program (which connects to the server implemented by server.c and server2.c), using Unit system calls for I/O
* client2.c &mdash; similar to client.c, but using `fdopen` and standard I/O functions
