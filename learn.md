---
layout: projdefault
projectname: Reactors.IO
projectpath: 
logoname: reactress-mini-logo-flat.png
title: Reactors.IO Documentation Resources
permalink: /learn/index.html
---


Here you can find various documentation sources for the Reactors framework.


## Getting started guide

The following step-by-step guide explains the concepts in the framework in detail,
and it is the best way to get started:

- [Reactors.IO Guide](http://reactors.io/tutorialdocs/reactors/)


## API docs

- [Reactors.IO 0.6 API](http://storm-enroute.com/apidocs/reactors/0.6/api)
- [Reactors.IO 0.5 API](http://storm-enroute.com/apidocs/reactive-collections/0.5/api)
- [Reactors.IO 0.4 API](http://storm-enroute.com/apidocs/reactive-collections/0.4/api)


## Talks and videos

These talks and videos cover different aspects of the Reactors.IO framework.

<table class="talks-papers">
<tbody>
<tr>
  <td>
    <a href="https://www.youtube.com/watch?v=7lulYWWD4Qo">
      <img height="48px" src="/resources/images/scala-days.png"/>
    </a>
  </td>
  <td>
    <a href="https://www.youtube.com/watch?v=7lulYWWD4Qo">
      <span class="talk-title">
        Reactors - Road to Composable Distributed Computing
      </span>
    </a>
    <br/>
    ScalaDays Berlin 2016
  </td>
</tr>
<tr>
  <td>
    <a href="https://www.youtube.com/watch?v=w8B4bJ1XV2E">
      <img height="48px" src="/resources/images/voxxed-zurich.png"/>
    </a>
  </td>
  <td>
    <a href="https://www.youtube.com/watch?v=w8B4bJ1XV2E">
      <span class="talk-title">
        Reactors - Programming Model for Composable Distributed Computing
      </span>
    </a>
    <br/>
    Voxxed Days Zurich 2016
  </td>
</tr>
</tbody>
</table>


## Reactors.IO Under-The-Hood

These documents describe the internal design and the goals behind
the Reactors.IO framework.

<table class="talks-papers">
<tbody>
<tr>
  <td>
    <a href="/resources/docs/reactors.pdf">
      <img height="48px" src="/resources/images/pdf.png"/>
    </a>
  </td>
  <td>
    <a href="/resources/docs/reactors.pdf">
      <span class="talk-title">
        Reactors, Channels and Event Streams for Composable Distributed Computing
      </span>
    </a>
    <br/>
    Reactor semantics and their role in distributed system design are described
    in detail in this Onward 2015 paper.
  </td>
</tr>
<tr>
  <td>
    <a href="/resources/docs/event-streams.pdf">
      <img height="48px" src="/resources/images/pdf.png"/>
    </a>
  </td>
  <td>
    <a href="/resources/docs/event-streams.pdf">
      <span class="talk-title">
        Containers and Aggregates, Mutators and Isolates for Reactive Programming
      </span>
    </a>
    <br/>
    Event streams and related event-based programming abstractions
    are described in this paper.
    At the time the paper was written,
    event streams used to have the <code>Reactive[T]</code> type
    (now represented with the <code>Events[T]</code> type).
    Observers used to have the type <code>Reactor[T]</code>
    (now represented with the <code>Observer[T]</code> type).
    This was changed in the Reactors.IO 0.6 version.
  </td>
</tr>
</tbody>
</table>

<br/>
