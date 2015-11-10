---
layout: post
title: JVM and clock_gettime syscall
---

The applications I'm developing at work are latency sensitive. For many purposes, including profiling, we take current time in the JVM applications, by calling System.currentTimeMillis() and System.nanoTime(). Therefore, I'm curious what's happening in JVM when they are called.

Here's the simple code that I ran with strace.

{% highlight java %}
package info.intransient.sandbox;

public class CurrentTimeMillis {
  public static void main(String[] args) {
    System.out.println("currentTimeMillis: " + System.currentTimeMillis());
  }
}
{% endhighlight %}

{% highlight java %}
package info.intransient.sandbox;

public class NanoTime {
  public static void main(String[] args) {
    System.out.println("nanoTime: " + System.nanoTime());
  }
}
{% endhighlight %}

This strace command created a couple of files, one for each thread.

{% highlight bash %}
$ strace -ff -o currenttimemillis java -cp sandbox-1.0-SNAPSHOT.jar info.intransient.sandbox.CurrentTimeMillis
{% endhighlight %}

One of the files contain a string "currentTimeMillis",

<pre>
clock_gettime(CLOCK_MONOTONIC, {614330, 176432161}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176477861}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176507352}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176527802}) = 0
gettimeofday({1447051867, 250445}, NULL) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176667726}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176694005}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176714203}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176761911}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176798974}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176828951}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176854814}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176875236}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 176951019}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177025355}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177085764}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177119123}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177179695}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177215084}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177246442}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177280365}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177357411}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177401129}) = 0
clock_gettime(CLOCK_MONOTONIC, {614330, 177445675}) = 0
write(1, "currentTimeMillis: 1447051867250", 32) = 32
</pre>

So apparently it calls gettimeofday. There are two items, however, that I don't understand.

- What are the other clock_gettime?
- This ran on Linux kernel of the version that supports vDSO. Does this still really issue the system call?

I will look into them in following posts.

