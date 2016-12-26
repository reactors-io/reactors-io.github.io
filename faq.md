---
layout: projdefault
projectname: Reactors.IO
projectpath: 
logoname: reactress-mini-logo-flat.png
title: Frequently Asked Questions
permalink: /faq/index.html
---


This sections covers some of the frequently asked questions about Reactors.IO.
Some of these questions were frequently raised on social media and mailing lists,
for others, we felt that they are important to address here.


### Can I still write my traditional actor programs?

Yes.
the reactor programming model generalizes the traditional actor model.
This means that all the programs representable in the actor model
must be representable with reactors.

Here is an example in Scala
(bindings for Java also exist).
We declare the `LegacyActor` that can receive events of any type.
We then use its main event source to listen for incoming events,
and match based on the event type.
If we receive an integer, we print it,
and if we receive a string `"terminate"`,
we terminate the reactor by sealing its event source.
This is shown in the following:

```scala
class LegacyActor extends Reactor[Any] {
  main.events onMatch {
    case n: Int => println("Number " + n)
    case "terminate" => main.seal()
  }
}
```

Above `Reactor[Any]` is an equivalent of a traditional actor,
and `main.events` is the equivalent of a traditional `receive` statement.
To start an instance of this reactor:

```scala
val ch: Channel[Any] = system.spawn(Proto[LegacyActor])
```

The channel returned by `spawn` is the alternative
of the **actor reference** (also called **process ID**)
We can use it to send messages to the reactor,
just like in the actor model:

```scala
ch ! "terminate"
```

If you do this, you will not benefit from the composability and type-safety
that the Reactors.IO framework has to offer,
but you will be able to write all your legacy actor programs
just like you did before.


### Does Reactors.IO implement the Reactive Streams standard?

**Short answer:**

Reactors.IO is an actor framework, not a streaming framework,
so it has no concept of concurrent stream processing.
However,
you could use Reactors.IO to implement a streaming framework,
which can comply to that standard.

**Long answer:**

No, Reactors.IO does not implement the Reactive Streams standard.
Reactors.IO is not a concurrent streaming processing framework,
it is an actor framework. Incidentally, inside each "reactor",
there are single-threaded event streams
that are actually a first-class generalization of the "receive" statement.
The event streams support functional combinators used to coordinate
between events coming from different sources.

To draw a parallel with a traditional actor system,
a **channel** is an analogue of an **actor reference** or a **process ID**.
In a traditional actor system, you would do the following to send a message:

```scala
def example(ref: ActorRef) = ch ! "Hello"
```

In Reactors, you do:

```scala
def example(ch: Channel[String]) = ch ! "Hello"
```

The **event stream** is an analogue of the **receive statement**.
While in a traditional actor system, you do:

```scala
receive {
  case s: String => println(s)
}
```

In Reactors, you do:

```scala
main.events.onEvent { s =>
  println(s)
}
```

Incidentally, the event streams, such as `main.events` above,
also support some functional combinators,
used to transform and coordinate events comming from different event streams.
For example, you can do:

```scala
main.events.map(_.length).onEvent { len =>
  printlnt("Received string with length: " + len)
}
```

First-class event stream composition is important,
since it allows composing different protocols.
However, this does not make Reactors.IO a stream processing framework,
so we felt it is not necessary to implement the Reactive Streams standard.
You could, however, use the Reactors.IO to implement
a Reactive Streams-compliant stream processing framework,
in a similar way that Akka was used to implement Akka Streams.


### Akka Streams needed automatic backpressure, does Reactors have it?

**Short answer:**

Not by default, because it is not required.
But backpressure is provided in the
[`reactors-protocol` module](https://github.com/reactors-io/reactors/blob/master/reactors-protocol/shared/src/main/scala/io/reactors/protocol/backpressure-protocols.scala).

**Long answer:**

Backpressure was a concern for Akka Streams because stream processing is distributed
across different concurrency units. Here backpressure prevents an upstream
event-processing node to send too many events to a downstream event-processing node,
and overflow it. This is what one typically has to do in a stream processing framework.
In the event streams used inside the Reactors framework, the event-stream chain is
always inside a single reactor. Inside a single reactor, events are propagated so
one-after-another, and an upstream event stream waits until the downstream event stream
processes the event. For example:

```scala
val emitter = new Events.Emitter[Int]
val mapped = emitter.map(_ * 2)
mapped.onEvent(println)
for (i <- 0 until 5) emitter.react(i)
```

Here we define an event stream chain that looks like this:

```scala
[emitter] -> [mapped] -> [onEvent]
```

When we enter the loop, and the first event gets is emitted in the `emitter.react` call,
the loop effectively stops, and event propagation starts. The value `0` is passed to
`mapped`, which multiplies it by `2`, and then to `onEvent`, which prints it to the
standard output. Control then returns to the loop, which only then emits `1`.

The "producer" (i.e. the loop) here waits for the event propagation to complete before
sending the next event. For this reason, the producer cannot overflow the consumer, and
backpressure is not necessary.

On the other hand, it is possible that there are two independent reactors,
`Producer` and `Consumer`, and that the `Producer` reactor is sending events over a
channel to Consumer (to draw a parallel with a classical actor system, you have to
actors, and the first is using an actor reference of the second to send messages).
The `Producer` reactor could be too fast, and overflow the second reactor.
In this case, backpressure is necessary (although note, that a similar problem exists
in classical actor systems, and that they don't give much to solve the problem for you -
you need to implement backpressure itself).

The good thing is, it is fairly straightforward to implement a `BackPressureChannel`
in the Reactors framework, which, when established between two reactors,
always ensures that the `Producer` does not overflow the `Consumer`.
Such a modular protocol is included in the
[`io.reactors.protocol` package](https://github.com/reactors-io/reactors/blob/master/reactors-protocol/shared/src/main/scala/io/reactors/protocol/backpressure-protocols.scala).
