---
layout: default
title: "Lab 13: Network Arithmetic Server"
---

# Getting Started

# Your Task

Write a server that

-   listens on a TCP port and accepts connections
-   reads a single line of text from each connection
-   parses the received text to extract two integer values
-   computes the sum of the integer values
-   sends a single line of text containing the sum back to the client
-   closes the connection socket

You can use the socket example programs from [Lecture 15](../lectures/lecture15.html) as a reference.

# Hints

* Use `fdopen` to convert the client socket into a FILE\* file handle
* Use `fscanf` to read two integer values from the client file handle
* Use `fprintf` to send the result value back to the client

# Testing

You can use the **telnet** program to test the server. For example, if the server is listening on port 10001, then run the command

    telnet localhost 10001

to connect to the server. You can type a line of text containing the two numbers to be added. You should see a response containing their sum.
