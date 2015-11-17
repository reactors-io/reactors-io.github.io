---
layout: projdefault
projectname: Reactors.IO
projectpath: 
logoname: reactress-mini-logo-flat.png
title: Signals
permalink: /docs/0.4/signals/index.html
reactressversion: 0.4
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

    val emitter = Reactive.Emitter[Int]
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
