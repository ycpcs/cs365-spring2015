---
layout: default
title: "Lecture 15: Socket programming in C"
---

# Sockets

## TCP/IP Socket programming

Sockets - the Unix standard API for network I/O

> Windows has something almost identical, called "winsock"

## File descriptors

A file descriptor is an integer value naming an underlying open file resource.

File descriptors are used in all Unix system calls which do I/O, or other opertions on files/streams.

### read and write

**read** - read data from a file/stream named by a file descriptor into a buffer

{% highlight cpp %}
ssize_t read(int fd, void *buf, size_t count);
{% endhighlight %}

**write** - write data from a buffer to a file/stream named by a file descriptor

{% highlight cpp %}
ssize_t write(int fd, const void *buf, size_t count);
{% endhighlight %}

[demo programs - read user's name, write to file, other program reads from file]

**fdopen**

> read and write are a pain. For example, no formatted input/output.
>
> Solution: convert a file descriptor into a FILE \* using the fdopen() function.  Then, use standard I/O functions such as `fprintf`, `fgets`, and `fscanf` to do I/O

Note that attempting to use a single FILE\* to read from and write from the pipe will not necessarily work.  (TODO: figure out why!)  A work-around is to use the `dup` system call to duplicate the socket file descriptor, and then make two calls to `fdopen` to create file handles for reading an writing:

{% highlight c %}
int fd = socket(...);

FILE *read_fh = fdopen(fd, "r");
FILE *write_fh = fdopen(fd, "w");
{% endhighlight %}

## Unix man pages

The Unix `man` command shows Unix manual pages.  These are extremely useful for getting quick reference documentation on system calls and C library functions.

Unix system calls (such as **read** and **write**) are documented in section 2 of the manual, and standard C library functions (such as **fdopen**) are documented in section 3 of the manual.

Some example commands to try:

    man 2 open
    man 2 read
    man 2 write
    man 3 fdopen

## Sockets

A socket is a *communications endpoint*.

Datagram socket: Can send/receive *datagrams* - discrete chunks of data. Analogy: mailbox.

Stream socket: is one end of a *connection*. A connection is a private communication channel between two communicating processes. Analogy: telephone.

Important concepts:

> Address - uniquely identifies a *host* on a network
>
> Port - distinguishes sockets on the same host from each other.

At the time the socket API was created (at UC Berkeley in the early 1980s), a variety of network protocol stacks were in active use, such as

-   TCP/IP
-   DECNET
-   others?

For this reason, the socket API treats socket addresses as an opaque data type, with multiple address types as "subclasses".

sockaddr\_un - a union of all available socket types

## Socket API

The socket API is implemented in a number of system calls.

socket - create a socket

{% highlight cpp %}
int socket(int domain, int type, int protocol);
{% endhighlight %}

domain - what protocol family the socket will use. PF\_INET for TCP/IP

type - SOCK\_STREAM for stream, SOCK\_DGRAM for datagram

protocol - identifies a particular protocol to use, if not uniquely determined by type. Just use 0 for any TCP/IP socket.

## Example programs

Demo programs:

> [socket.zip](socket.zip)

* write\_to\_file.c &ndash; open a file and write to it using Unix system calls
* server.c &ndash; a simple server program implemented using Unix system calls for I/O
* server2.c &ndash; similar to server.c, but using `fdopen` and standard I/O functions
* client.c &ndash; a simple client program (which connects to the server implemented by server.c and server2.c), using Unit system calls for I/O
* client2.c &ndash; similar to client.c, but using `fdopen` and standard I/O functions
