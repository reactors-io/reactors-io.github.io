---
layout: projdefault
projectname: Reactors.IO
projectpath: 
logoname: reactress-mini-logo-flat.png
title: Download
permalink: /download/index.html
---




Reactors.IO is hosted on Sonatype OSSRH.
You can download the JAR directly,
or add Reactors.IO to your SBT file as a managed dependency.


## Direct

### Scala 2.11

<a href='http://search.maven.org/remotecontent?filepath=com/storm-enroute/reactors_2.11/{{ site.reactors_version }}/reactors_2.11-{{ site.reactors_version }}.jar'>
  <img class="buildstatus" src='https://img.shields.io/maven-central/v/com.storm-enroute/reactors_2.11.svg' onerror='this.style.display="none"' />
</a>

- latest stable version: [{{ site.reactors_version }}](http://search.maven.org/remotecontent?filepath=com/storm-enroute/reactors_2.11/{{ site.reactors_version }}/reactors_2.11-{{ site.reactors_version }}.jar)
- snapshot version: [{{ site.reactors_snapshot_version }}](https://oss.sonatype.org/service/local/artifact/maven/redirect?r=snapshots&g=com.storm-enroute&a=reactors_2.11&v={{ site.reactors_snapshot_version }}&e=jar)


### Scala 2.10

<a href='http://search.maven.org/remotecontent?filepath=com/storm-enroute/reactors-core_2.10/{{ site.reactors_version }}/reactors-core_2.10-{{ site.reactors_version }}.jar'>
  <img class="buildstatus" src='https://img.shields.io/maven-central/v/com.storm-enroute/reactors-core_2.10.svg' onerror='this.style.display="none"' />
</a>

- latest stable version: [{{ site.reactors_version }}](http://search.maven.org/remotecontent?filepath=com/storm-enroute/reactors-core_2.10/{{ site.reactors_version }}/reactors-core_2.10-{{ site.reactors_version }}.jar)


## SBT

To add Reactors.IO to you project,
add the following lines to your SBT build definition file (e.g. `build.sbt`):

    resolvers ++= Seq(
      "Sonatype OSS Snapshots" at
        "https://oss.sonatype.org/content/repositories/snapshots",
      "Sonatype OSS Releases" at
        "https://oss.sonatype.org/content/repositories/releases"
    )

    libraryDependencies ++= Seq(
      "com.storm-enroute" %% "reactors" % "{{ site.reactors_version }}")

If you want to live on the cutting edge,
you can use the nightly snapshot by adding the following line instead:

    libraryDependencies ++= Seq(
      "com.storm-enroute" %% "reactors" % "{{ site.reactors_snapshot_version }}")
