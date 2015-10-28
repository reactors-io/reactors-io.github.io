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

The `MyIso` isolate is not very interesting because it does not receive any events.
In the next example, we declare a `HiIso`, which can receive `String` events.
The `HiIso` isolate adds an `onMatch` event handler in the constructor,
which reacts to incoming `"Hi"` events:

    class HiIso extends Iso[String] {
      main.events onMatch {
        case "Hi" =>
          println("Howdee!")
          main.seal()
      }
    }

After the first `"Hi"` event arrives, `HiIso` prints `"Howdee!"`,
and terminates by sealing its main channel.

Every isolate `Iso[T]` can have an unlimited number of typed input channels,
but is created with only two channels -- the `main` channel,
which accepts events of type `T`,
and the system channel that accepts special `SysEvent` objects.
We will learn more about isolate communication in a subsequent section.
Before we get to isolate communication, we will learn how to start isolates.


## Starting an Isolate

In the previous section, we saw how to declare an isolate.
Declaring an isolate does not actually start its execution --
the same isolate class can be reused many times to start an isolate instance.
To start an isolate, we need an existing isolate system.
An *isolate system* is an entity that groups isolates inside a single machine,
tracks their names, and can create new isolates when requested.
We will use the default isolate system factory method -- `IsoSystem.default`.


## Isolate Communication

