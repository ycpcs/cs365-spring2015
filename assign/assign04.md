---
layout: default
title: "Assignment 4: Research Project"
---

**Choose a project by Tuesday, April 14th**

**Code and report due Friday, May 8th**

Presentations during Final Exam time: **Tuesday, May 12th, 12:45-2:45**

Your Task
=========

You may work individually or in groups of 2 or 3.

Research a topic in parallel or distributed computing. Write a program to demonstrate the technique or algorithm you have researched. Run experiments to analyze the performance/behavior of the program. Write a report summarizing your findings. Give an oral presentation in to present your findings to the rest of the class.

You should discuss your project idea with me no later than **Tuesday, April 14th**.

General Topic Ideas
===================

-   Find a parallel algorithm and implement it using pthreads or MPI
-   Implement a distributed system (in which processes communicate over a network) using TCP/IP
-   Design and implement a parallel algorithm using pthreads or MPI

Some Specific Ideas
===================

-   Implement a strategy game such as Checkers or Go using a parallel state space search to consider possible moves.
-   Implement and benchmark a lock-free data structure using atomic machine instructions. See Michael Scott's [High-performance synchronization](http://www.cs.rochester.edu/wcms/research/systems/high_performance_synch/) web page.
-   Use GPGPU computation to implement a parallel algorithm.
-   Implement a 2-processor merge algorithm where one thread starts at the beginning of the arrays being merged, and the second thread starts at the end. Is it possible to make this faster than a sequential merge?

It is fine for multiple people/groups to work on the same problem.

Submitting
==========

Upload a zip file containing your code and your report to the Marmoset server as **assign04**:

> <https://cs.ycp.edu/marmoset/>
