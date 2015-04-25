---
title: "<define> task for NAnt"
date: 2008-01-24T15:42:02.000Z
x-drupal-nid: 210
x-needs-review: 2008-01-24T15:42:02.000Z
---
Essentially, you write a new task like this:

<pre><define name="echo3">
  <echo message="${this.message}"/>
  <echo message="${this.message}"/>
  <echo message="${this.message}"/>
</define></pre>

...and then you call it like this:

<pre><echo3 message="Hello World"/></pre>

Any parameter passed to the defined task is available as this.foo inside the defined task. I've found it useful when you don't want to write a task in C#, perhaps because all you're doing is calling a bunch of other NAnt tasks.