---
layout: projdefault
projectname: Reactors.IO
projectpath: 
logoname: reactress-mini-logo-flat.png
title: Event Streams
permalink: /docs/0.6/events/index.html
reactressversion: 0.6
section: General
pagenum: 1
pagetot: 10
---





The basic data-type that drives most computations in Reactors.IO is
called an **event stream**, represented by the type `Events[T]`.
An event stream represents an entity
that can occasionally produce values of type `T`.
These values are called *events*.

An event stream is entirely a single-threaded entity.
There is no concurrency associated with an event stream --
it can only propagate events on one thread at a time.
A programmer need not worry about an event concurrently being
emitted on the same event stream.
This simplifies the programming model, makes programs more maintainable
and ensures better performance.
As we will see, it also allows to encapsulate mutable objects
inside event stream.

In what follows, we describe event streams and their different variants.
We show how to compose event streams in a declarative way
to yield concise and understandable reactive programs.


## Event Streams

The `Events[T]` type is represented with the following trait:

    trait Events[+T] {
      // lots of stuff
    }

To react to events produced by the event stream,
we need to **subscribe** to it.
One way to do this is to use the `onReaction` method of the `Events[T]` trait:

    def onReaction(reactor: Reactor[T]): Events.Subscription

The `onReaction` method takes a reactor of type `Reactor[T]` and returns a
`Subscription` object.
A `Reactor[T]` is a type that provides two methods called `react` and `unreact`:

    trait Reactor[-T] {
      def react(value: T): Unit
      def unreact(): Unit
    }

Whenever an event stream produces an event, the reactors,
which were previously subscribed
to the event stream with `onReaction`,
are notified -- their `react` method is invoked with the `value` of the event.

Every event stream can at some point decide to stop emitting events.
We say that the event stream **unreacts**.
At this point, the `unreact` value is called on all its reactors.

The `Subscription` object returned by `onReaction`
represents the subscription by the specific reactor.
It has a single method `unsubscribe`.
If the `unsubscribe` method is invoked,
events are no longer passed to the corresponding reactor
and its `unreact` method is never called.

Reactors.IO offers several more Scala-idiomatic ways to subscribe to events.
The `onEvent` method is used to subscribe to events, but ignore unreactions.
The `onMatch` method is similar, but takes a partial function --
only the events for which the partial function is defined are considered.
The `on` method executes a block of code when an event arrives --
it is most appropriate for event streams of type `Events[Unit]`.
Finally, the `onUnreact` method executes a block of code when the event stream
unreacts.

    def onEvent(reactor: T => Unit): Events.Subscription
    def onMatch(reactor: PartialFunction[T, Unit]): Events.Subscription
    def on(reactor: =>Unit): Events.Subscription
    def onUnreact(reactor: =>Unit): Events.Subscription

There are several basic concrete event streams:

- `Events.Never` -- never emits any events
- `Events.Emitter` -- emits an event every time its `+=` method is called


## Event Emitters

An **event emitter** is a concrete event stream.
We create it as follows:

    val emitter = new Events.Emitter[Int]

The events emitter produces an events each time an event is added to it by the
`react` method.
The following example prints `"Hello world!"` to the standard output.

    val printSubscription = emitter onEvent println
    emitter react "Hello world!"

After calling `unreact` on the emitter, the emitter unreacts.
Subsequent invocations of `react` are ignored.

    val unreactSubscription = emitter onUnreact {
      println("done for today.")
    }
    emitter.unreact()


## Subscriptions

A subscription is an object used to unsubscribe from a reaction.
In most cases this means that the corresponding entity will no longer receive
events.
We unsubscribe by calling the `unsubscribe` method:

    val printSubscription = emitter onEvent println
    emitter += "Hi!"
    printSubscription.unsubscribe()
    emitter += "You can't hear me!"

Note that we always store the `Subscription` object returned by the `onX`
methods.
Since Reactors.IO 0.6, we *do not* have to do this.
It is equally correct to write the following:

    emitter onEvent println

Every `onX` call is meant to have side-effects, so its subscription gets
automatically stored.
Be aware that without keeping a reference to the `Subscription` there is no way
to unsubscribe from the callback, so the callback could react forever,
leading to effects called **memory leaks** and **time leaks**.
If you plan to unsubscribe from the callback before the application ends, you
should store the subscription returned by `onEvent`, or any other `onX` method.

For all other combinators,
when Reactors.IO runtime detects that the program no longer has a reference to
the `Subscription`, it automatically unsubscribes.
This automatic unsubscription usually happens during the first subsequent GC cycle.
The `foreach` is an example of a combinator for which the subscription is not
automatically saved:

    emitter foreach println
    System.gc()
    emitter += "Chances are you won't hear me anymore."

This behaviour is by design.
Consider the following method `sumOfSquares` that uses an
emitter `squares` to compute a sum of squares of first `n` integers.
It subscribes to `squares` to update its `sum` and
uses an auxiliary method `emitSquare` to emit `n` squares:

    val squares = new Events.Emitter[Int]

    def emitSquare(x: Int) = squares += x * x

    def sumOfSquares(n: Int): Int = {
      var sum = 0
      
      squares.foreach(sum += _)
      for (i <- 1 until n) emitSquare(i)

      sum
    }

The scope in which the callback that updates `sum` is useful
is only the method `sumOfSquares`.
Once the method completes, this callback is not only useful,
but also dangerous.
It occupies a region of memory that cannot be freed and
captures the lifted version of the `sum` variable --
it leads to memory leaks.
Furthermore, whenever some other part of the program
emits an event on `squares`, the callback that updates
the `sum` must be executed and wastes computational resources --
in FRP, this is known as a time leak.

For these reasons, Reactors.IO subscribe to the
"If you didn't save it, you don't need it." philosophy.

<table class="docs-tip">
<td><img src="/resources/images/reactress-warning.png"/></td>
<td>
Always store a reference to the subscription of the callback
that is important in the program.
Otherwise, the callback is eventually automatically unsubscribed.
</td>
</table>


## Functional Composition on Event Streams

Event streams represent event sources and reactors are very similar to *observers*.
However, using only the `onX` family of methods on event streams soon
results in a *callback hell* -- in a program composed of a large number
of unstructured `onX` method calls understandibility becomes a problem.

To overcome this problem, we use *functional composition* on event streams --
a programming pattern in which more complex values are created by declaratively
composing simpler ones.

Let's rewrite the method `sumOfSquares` from before.
This time we do not use a callback and a mutable variable.
Instead, we use the `map` and `scanPast` combinators.
The `map` combinator transforms events in one event stream into events for a derived
event stream -- we use it to map each number into its square.
The `scanPast` combinator combines the last and the current event to produce a
new event for the derived event stream --
we use it to add the previous value of the sum to the current one.
For example, if an input event stream produces numbers `0`, `1` and `2`, the
event stream returned by `scanPast` produces numbers `0`, `1` and `3`. 

    def sumOfSquares(n: Int): Int = {
      val emitter = new Events.Emitter[Int]
      val sum = emitter.map(x => x * x).scanPast(0)(_ + _)
      for (i <- 0 until n) emitter += n
      sum()
    }

The `Events[T]` trait comes with a large number of predefined combinators.
Same as with callbacks, you must always store the return value of the
combinator.
Not doing so eventually results in an automatic unsubscription.

A bunch of event stream values composed using functional combinators
forms a **dataflow graph**.
Emitters are source nodes in this graph, event streams obtained by various
combinators are inner nodes
and callback methods like `onEvent` form sink nodes.
Some combinators like `union` take several input event streams.
Such event streams correspond to nodes with multiple input edges in the dataflow
graph.

    val numbers = new Events.Emitter[Int]
    val even = numbers.filter(_ % 2 == 0)
    val odd = numbers.filter(_ % 2 == 1)
    val numbersAgain = even union odd


## Higher-order Event Streams

In some cases event streams produce events that are themselves event
streams -- we call them **higher-order event streams**.
A higher-order event stream can have a type like:

    Events[Events[T]]

There are multiple ways to access the events of type `T` from the inner event stream
values.
We might be interested in the events from the last `Events[T]` produced in the
higher-order event streams.
To access them, we use the `mux` operator -- this operator multiplexes events
from the last `Events[T]`:

    val higherOrder = new Events.Emitter[Events[Int]]
    val evens = new Events.Emitter[Int]
    val odds = new Events.Emitter[Int]
    val flattened: Events[Int] = higherOrder.mux
    val prints = flattened onEvent println

    evens += 2
    odds += 1
    higherOrder += evens
    odds += 3
    evens += 4 // prints 4
    higherOrder += odds
    evens += 6
    odds += 5 // prints 5

In some cases we want to obtain all the events from all the event stream
produced by the higher-order event stream.
To do so, we use the `union` combinator.

    val flattened: Events[Int] = higherOrder.union
   
    higherOrder += evens
    odds += 3
    events += 4 // prints 4
    higherOrder += odds
    evens += 6 // prints 6
    odds += 5 // prints 5




