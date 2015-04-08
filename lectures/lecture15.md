---
layout: default
title: "Lecture 15: Socket programming in C"
---

# Sockets

Demo programs:

> [socket.zip](socket.zip)

## TCP/IP Socket programming

Sockets - the Unix standard API for network I/O

> Windows has something almost identical, called "winsock"

## File descriptors

A file descriptor is an integer value naming an underlying open file resource.

File descriptors are used in all Unix system calls which do I/O, or other opertions on files/streams.

### read and write

read - read data from a file/stream named by a file descriptor into a buffer

{% highlight cpp %}
ssize_t read(int fd, void *buf, size_t count);
{% endhighlight %}

write - write data from a buffer to a file/stream named by a file descriptor

{% highlight cpp %}
ssize_t write(int fd, const void *buf, size_t count);
{% endhighlight %}

[demo programs - read user's name, write to file, other program reads from file]

**fdopen**

> read and write are a pain. For example, no formatted input/output.
>
> Solution: convert a file descriptor into a FILE \* using the fdopen() function.
>
> [convert demo programs: use formatted I/O to make things easier]

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
