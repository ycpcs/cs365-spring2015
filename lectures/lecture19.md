---
layout: default
title: "Lecture 19: Concurrency in Clojure and Erlang"
---

[Clojure](http://clojure.org) and [Erlang](http://www.erlang.org) are functional languages that have built-in support for concurrency.

Because they are functional languages (employing a programming model where most data structures are immutable), they allow the programmer to avoid some common types of errors (e.g., race conditions) that are common when using concurrency in imperative languages.

# Concurrency in Clojure

Example computation: the [Mandelbrot Set](http://en.wikipedia.org/wiki/Mandelbrot_set).

Here is a very basic sequential implementation of the Mandelbrot set computation.  The goal is to compute a grid of iteration counts, where each item in the grid is the number of times the equation

> Z = Z<sup>2</sup> + C

can be iterated before the magnitude of Z reaches 2, where C is an arbitrary complex number, Z is initially 0+0i, and the number of iterations is subject to a maximum number of iterations.  If the maximum iteration count is reached, then C is assumed to be in the Mandelbrot set.

{% highlight clojure %}
;; ------------------------------------------------------------
;; Complex numbers
;; ------------------------------------------------------------

; Ensure that complex numbers use doubles to represent the
; real and imaginary components.
(defn make-complex [r i]
  [(double r) (double i)])

(defn complex-add [[a b] [c d]]
  [(+ a c) (+ d b)])

(defn complex-mul [[a b] [c d]]
  [(- (* a c) (* b d)) (+ (* b c) (* a d))])

(defn complex-magnitude [[r i]]
  (Math/sqrt (double (+ (* r r) (* i i)))))

;; ------------------------------------------------------------
;; Mandelbrot set computation
;; ------------------------------------------------------------

; Starting from z = 0+0i, iterate z = z^2 + c until the magnitude
; of z reaches 2 or a maximum iteration count is reached.
(defn mandelbrot-iter [c maxiters]
  (loop [z (make-complex 0 0)
         count 0]
    (if (or (> (complex-magnitude z) 2) (>= count maxiters))
      count
      (recur (complex-add (complex-mul z z) c) (+ count 1)))))

; Compute a single row of iteration counts, starting from
; given real/imaginary values, incrementing real values by r-incr,
; generating n iteration counts, using maxiters as the maximum number
; of iterations.
(defn mandelbrot-compute-row [r i r-incr n maxiters]
  (mapv (fn [index]
          (mandelbrot-iter (make-complex (+ r (* r-incr index)) i) maxiters))
        (range 0 n)))

; Sequential computation, returns a sequence of rows, where
; each row is a sequence of iteration counts as computed
; by mandelbrot-compute-row.  Computes every n'th row starting
; from startrow.
(defn mandelbrot-compute-grid-seq [rmin imin rmax imax ncols nrows maxiters startrow n]
  (let [r-incr (/ (- rmax rmin) ncols)
        i-incr (/ (- imax imin) nrows)]
    (mapv (fn [row]
            (mandelbrot-compute-row rmin (+ imin (* row i-incr)) r-incr ncols maxiters))
          (range startrow nrows n))))

; Sequential computation, computing every row from 0 to nrows-1.
(defn mandelbrot-compute-grid-seq-all [rmin imin rmax imax ncols nrows maxiters]
  (mandelbrot-compute-grid-seq rmin imin rmax imax ncols nrows maxiters 0 1))
{% endhighlight %}

The `mandelbrot-compute-grid-seq-all` function performs a purely sequential computation.  On my computer, the call

{% highlight clojure %}
(mandelbrot-compute-grid-seq-all -2 -2 2 2 200 200 1000)
{% endhighlight %}

takes about 1.8 seconds to complete.

## Futures

One of the simplest and most useful concurrency constructs in Clojure is the *future*.  The `future` function takes a series of arbitrary expressions, returns a future.  A future is "potential" value: its value may not be known at the current time, but will be known at some point in the future.  A value of a future can be determined using the `deref` function.  If the future's value is known, then deref returns it.  If the future's value is not known yet (because its thread is still not finished), then `deref` waits until the value is ready.  The eventual value of a future is the value of the last evaluated expression.  Most futures only evaluate a single expression, since only one result can be produced by the future.

Futures support a very straightforward style of fork/join parallelism.  For example, here is a parallel implementation of the Mandelbrot set computation that creates one future per row of the grid:

{% highlight clojure %}
; Parallel computation using one future per row.
(defn mandelbrot-compute-grid-par-futures [rmin imin rmax imax ncols nrows maxiters]
  (let [r-incr (/ (- rmax rmin) ncols)
        i-incr (/ (- imax imin) nrows)
        row-fn (fn [row]
                 (future (mandelbrot-compute-row rmin (+ imin (* row i-incr))
                                                 r-incr ncols maxiters)))
        future-results (mapv row-fn (range 0 nrows))]
    (mapv deref future-results)))
{% endhighlight %}

On my computer, which has 3 CPU cores, the function call

{% highlight clojure %}
(mandelbrot-compute-grid-par-futures -2 -2 2 2 200 200 1000)
{% endhighlight %}

takes about 1 second to complete.

Because each future creates a new thread, and because thread creation and cleanup adds overhead to the computation, it may be helpful to create a fixed number of futures, and allow each future to work on a larger chunk of the overall problem.  For example, here is an "interleaved" version of the parallel Mandelbrot set computation, in which the number of futures to be created is specified as a parameter *n*, and each future computes every *n*th row.  The results computed by the futures (once waited for using `deref`) are then combined into a single sequence using the `interleave` function.

{% highlight clojure %}
; Parallel computation using a fixed number of futures.
; The computation performed by each future handles every n'th
; row (by making a call to mandelbrot-compute-grid-seq).
; This is a relatively efficient way to split up the
; computation over a fixed number of threads.
(defn mandelbrot-compute-grid-par-futures-interleaved [rmin imin rmax imax ncols nrows maxiters nthreads]
  (let [every-nth-row-fn (fn [j]
                           (future (mandelbrot-compute-grid-seq rmin imin rmax imax ncols nrows maxiters j nthreads)))
        partial-results (mapv every-nth-row-fn (range 0 nthreads))]
    (apply interleave (mapv deref partial-results))))
{% endhighlight %}

On my computer (3 cores) the function call

{% highlight clojure %}
(mandelbrot-compute-grid-par-futures-interleaved -2 -2 2 2 200 200 1000 3)
{% endhighlight %}

takes about .75 seconds.

## Atoms

Yeah.

## Agents

Uh-huh.

# Concurrency in Erlang

We will probably do this next week.

<!-- vim:set wrap: ­-->
<!-- vim:set linebreak: -->
<!-- vim:set nolist: -->
