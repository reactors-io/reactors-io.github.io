---
layout: projdefault
projectname: Reactive Collections
projectpath: 
logoname: reactress-mini-logo-flat.png
title: Signals
permalink: /docs/0.6/signals/index.html
reactressversion: 0.6
section: General
pagenum: 2
pagetot: 10
---



Signals are special kind of reactive values that cache the last produced event.

    trait Signal[T] {
      def apply(): T
    }

To obtain the value of the last event, we call the signal's `apply` method.
Any reactive value can be converted into a signal by calling its `signal` method.
We need to provide the initial value of the signal when calling this method.

    val emitter = new Reactive.Emitter[Int]
    val sig = emitter.signal(0)

The signal `sig` can then be queried to find out about the last emitted event:

    sig() // returns 0
    emitter += 1
    sig() // returns 1
    emitter += 5
    sig() // returns 5

Signals support some additional combinators, such as `zip`:

    (Signal.Const(10) zip sig()) {
      _ + _
    }


## Aggregate signals

A *static aggregate signal* is an aggregation of the values from several signals.
As soon as one of the signals in the aggregation changes, the aggregate signal is updated:

    val s0 = Signal.Const(1)
    val e1 = new Reactive.Emitter[Int]
    val s1 = e1.signal(0)
    val e2 = new Reactive.Emitter[Int]
    val s2 = e1.signal(2)
    val a = Signal.Aggregate(s0, s1, s2)
    a() // returns 3
    s2 += 5
    a() // returns 6

Importantly, the aggregate signal is updated in `O(log n)` time,
where `n` is the number of signals it depends on.


## Mutable signals

A *mutable signal* is used to wrap a mutable object.
Upon mutating the wrapped mutable object,
the signal's `onMutated` method must be called:
    
    import scala.collection._
    val buffer = mutable.ArrayBuffer[Int]()
    val bs = Signal.Mutable(buffer)
    val logs = bs.onEvent(println)
    buffer += 1
    bs.onMutated()

Calling `onMutated` is cumbersome,
so we mainly use mutable signals with the `mutate` combinator on reactives:

    import scala.collection._
    val bs = Signal.Mutable(mutable.ArrayBuffer[Int]())
    val logs = bs.onEvent(println)

    val emitter = new Reactive.Emitter[Int]
    val mutations = emitter.mutate(bs) { num =>
      bs() += num
    }



