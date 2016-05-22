---
layout: projdefault
projectname: Reactors.IO
projectpath: 
logoname: reactress-mini-logo-flat.png
title: Whitepaper
permalink: /under-the-hood/index.html
---

Event streams and related abstractions for event-based programming
are described in
[this paper](/resources/docs/event-streams.pdf).
Note that at the time the paper was written,
event streams (now represented with the `Events[T]` type)
used to have the `Reactive[T]` type,
and observers (now represented with the `Observer[T]` type)
used to have the type `Reactor[T]`.
This was changed in the Reactors.IO 0.6 version.

Reactor semantics and their role in distributed system design
are described in detail in
[this Onward 2015 paper](/resources/docs/reactors.pdf).
