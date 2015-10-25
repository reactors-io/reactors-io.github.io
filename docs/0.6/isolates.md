---
layout: projdefault
projectname: Reactive Collections
projectpath: 
logoname: reactress-mini-logo-flat.png
title: Isolates
permalink: /docs/0.6/isolates/index.html
reactressversion: 0.6
section: General
pagenum: 5
pagetot: 10
---



## Defining an Isolate

An isolate is defined by declaring a class that extends `Iso[T]` for some type `T`.
The type `T` is the type of events that an isolate can receive on its main
channel.
Here is an example:

    class MyIso extends Iso[String] {
      main.seal()
    }

The `MyIso` isolate does not do anything too smart -- as soon as it runs,
it seals its main channel, meaning that no additional events will be received.
Once the main channel is sealed, the `MyIso` isolate dies.


## Starting an Isolate


## Isolate Communication

