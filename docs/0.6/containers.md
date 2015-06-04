---
layout: projdefault
projectname: Reactive Collections
projectpath: 
logoname: reactress-mini-logo-flat.png
title: Containers
permalink: /docs/0.6/containers/index.html
reactressversion: 0.6
section: General
pagenum: 4
pagetot: 10
---


In addition to reactive values and signals described earlier,
the Reactive Collections framework also provides data collections
whose contents can change as a result of certain events,
or whose mutations may trigger arbitrary sets of events.
Such data collections are called *reactive containers*
and are described in this section.

Similar to the reactive values and signals,
reactive containers are currently specialized
for primitive types `Int`, `Long` and `Double`.


## Reactive cells

Similar to how functional languages model mutable memory locations using *mutable cells*,
a reactive programming model can provide reactive cells,
whose mutations trigger events.
In Reactive Collections,
the reactive cell abstraction is represented by the `RCell[T]` type,
whose simplified interface is as follows:

    class RCell[T] extends Signal[T] {
      def :=(x: T): Unit
    }

Calling `apply` returns the previous value associated with the `RCell`,
and calling `:=` associates a new value with the reactive cell.
Each reactive cell is simultaneously a signal,
so you can use functional combinators to transform `RCell` objects.

In the following, we create a reactive cell that represents the number of elapsed seconds,
and use it to define the number of elapsed hours and elapsed minutes,
as well as the number of minutes elapsed in the current hour (`currentMinute`):

    val elapsedSeconds = RCell[Long](0L)
    val elapsedMinutes = seconds.map(_ / 60)
    val elapsedHours = minutes.map(_ / 60)
    val currentMinute = (elapsedMinutes zip elapsedHours) {
      _ - _ * 60
    }

When we assign a new number of elapsed seconds to the `elapsedSeconds` reactive cell,
the rest of the signals are automatically updated with new values.
The signals `elapsedMinutes`, `elapsedHours` and `currentMinute`
are *constrained* to the `elapsedSeconds` value.

Reactive cells are the simplest form of reactive containers.
In the next section,
we study how to use reactive maps.


## Reactive maps

Reactive map is the reactive counterpart of the mutable map abstraction.
It is represented with the `RMap[K, V]` type.
The reactive map is the first true reactive container we encounter.
It has roughly the following interface:

    trait RMap[K, V <: AnyRef]
    extends RContainer[(K, V)] {
      def apply(k: K): V
      def applyOrNil(k: K): V
      def nil: V
      def get(k: K): Option[V]
      def react: RMap.Lifted[K, V]
    }

Calling `apply` with a key `K` returns the value `V` associated with that key,
or throws an exception if no such key is in the map.
Calling `applyOrNil` with a key `K` does the same if the key is in the map,
but instead returns a special *nil* value if the key is not in the map.
The nil value is defined for the type `V` when the reactive map is created,
and can be queried by calling the `nil` method.
For the `RMap` type, the `nil` method usually just returns `null`.
The `applyOrNil` method is useful when we want to get the value,
but do not want to catch exceptions or allocate any objects.
Finally, if object allocation is not a concern,
there is the good old `get` method, which returns an `Option[V]` object,
and should be known to you from the standard Scala collection API.

The `RMap[K, V]` type also extends the `RContainer` type,
which specifies additional functionality.
We will study the `RContainer` type in a subsequent section.

<table class="docs-tip">
<td><img src="/resources/images/reactress-warning.png"/></td>
<td>
Note that the <code>RMap</code> type is specialized at the key type <code>K</code>,
and is meant to store reference values <code>V</code>.
To store primitive values, use the <code>RHashValMap[K, V]</code> type,
and note that in the Scala 2.11.x both the <code>K</code> and <code>V</code> type
still need to be specialized to benefit from specialization
(this might eventually be resolved by either miniboxing
or JVM-level specialization).
</td>
</table>

The `react` method on the `RMap` type is the place where things get interesting.
This method returns the `RMap.Lifted[K, V]` view of the reactive map,
which has a similar interface as `RMap[K, V]`,
but whose methods return reactive values of type `Reactive[V]`
instead of plain scalar values of type `V`.

    object RMap {
      trait Lifted[K, V]
      extends RContainer.Lifted[(K, V)] {
        def apply(k: K): Reactive[V]
      }
    }

Calling `apply` on the lifted reactive map returns
a reactive value that emits all the values subsequently added to that key.
It does not matter if the key is already present in the map, or not.

Here is a simple example of how to use reactive maps:

    val stringReps = RMap[Int, String]()
    val prints = stringReps.react(1) onEvent {
      txt => println("String for 1: " + txt)
    }
    stringReps(1) = "1"    // prints: String for 1: 1
    stringReps(1) = "One"  // prints: string for 1: One

The default `RMap` implementation is the `RHashMap` class.
This reactive map is implemented with a closed-addressing hash table.

The reactive map also contains several projection methods:

      def entries: PairContainer[K, V]
      def keys: RContainer[K]
      def values: RContainer[V]

These methods make it simpler or more efficient
to work with keys and values in the map, or pairs of thereof.


## Reactive sets

Reactive sets are represented with the `RSet` trait and have the following interface:

    trait RSet[T] extends RContainer[T] {
      def +=(elem: T): Boolean
      def -=(elem: T): Boolean
      def apply(elem: T): Boolean
      def inserts: Reactive[T]
      def removes: Reactive[T]
    }

The first three methods should be familiar from the Scala standard library collections.
The `inserts` and `removes` methods return reactive values which emit the elements that
are inserted into or removed from the set, respectively.

The default reactive set implementation is `RHashSet`.


## Reactive containers

The reactive sets revealed a pair of interesting methods called `inserts` and `removes`.
These methods are present on both reactive maps and reactive sets,
as well as other reactive containers,
and are fundamental when implementing functional combinators on reactive containers. 

Reactive containers are represented with the `RContainer[T]` type.
The simplified `RContainer[T]` interfaces is as follows:

    trait RContainer[T] {
      def inserts: Reactive[T]
      def removes: Reactive[T]
      def react: RContainer.Lifted[T]
      def foreach(f: T => Unit): Unit
      def size: Int
    }

TODO


## Reactive queues


## Reactive aggregate containers



