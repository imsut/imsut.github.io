---
layout: post
title: clock_gettime in hotspot
---

JVM issues clock_gettime syscall quite often as we saw in [the last article](/2015/11/08/jvm-and-clock_gettime).
Digging into hotspot code to see where/why clock_gettime is called.
all code snippets are from [hotspot-87ee5ee27509](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/rev/87ee5ee27509).

`clock_init()` in os_linux.cpp checks if `clock_gettime()` is available, and assign its function pointer to `_clock_gettime` variable.

{% highlight c++ %}
void os::Linux::clock_init() {
  // we do dlopen's in this particular order due to bug in linux
  // dynamical loader (see 6348968) leading to crash on exit
  void* handle = dlopen("librt.so.1", RTLD_LAZY);
  if (handle == NULL) {
    handle = dlopen("librt.so", RTLD_LAZY);
  }

  if (handle) {
    int (*clock_getres_func)(clockid_t, struct timespec*) =
           (int(*)(clockid_t, struct timespec*))dlsym(handle, "clock_getres");
    int (*clock_gettime_func)(clockid_t, struct timespec*) =
           (int(*)(clockid_t, struct timespec*))dlsym(handle, "clock_gettime");
    if (clock_getres_func && clock_gettime_func) {
      // See if monotonic clock is supported by the kernel. Note that some
      // early implementations simply return kernel jiffies (updated every
      // 1/100 or 1/1000 second). It would be bad to use such a low res clock
      // for nano time (though the monotonic property is still nice to have).
      // It's fixed in newer kernels, however clock_getres() still returns
      // 1/HZ. We check if clock_getres() works, but will ignore its reported
      // resolution for now. Hopefully as people move to new kernels, this
      // won't be a problem.
      struct timespec res;
      struct timespec tp;
      if (clock_getres_func (CLOCK_MONOTONIC, &res) == 0 &&
          clock_gettime_func(CLOCK_MONOTONIC, &tp)  == 0) {
        // yes, monotonic clock is supported
        _clock_gettime = clock_gettime_func;
        return;
      } else {
        // close librt if there is no monotonic clock
        dlclose(handle);
      }
    }
  }
  warning("No monotonic clock was available - timed services may " \
          "be adversely affected if the time-of-day clock changes");
}
{% endhighlight %}

OS independent function `os::javaTimeNanos` calls `clock_gettime` here. Although there're some other call sites of `clock_gettime` in os_linux.cpp, I'll look into `os::javaTimeNanos` first.

{% highlight c++ %}
jlong os::javaTimeNanos() {
  if (Linux::supports_monotonic_clock()) {
    struct timespec tp;
    int status = Linux::clock_gettime(CLOCK_MONOTONIC, &tp);
    assert(status == 0, "gettime error");
    jlong result = jlong(tp.tv_sec) * (1000 * 1000 * 1000) + jlong(tp.tv_nsec);
    return result;
  } else {
    timeval time;
    int status = gettimeofday(&time, NULL);
    assert(status != -1, "linux error");
    jlong usecs = jlong(time.tv_sec) * (1000 * 1000) + jlong(time.tv_usec);
    return 1000 * usecs;
  }
}
{% endhighlight %}

On the assumption that the behavior of calling clock_gettime is OS independent, which may be wrong though, I grepped javaTimeNanos under "src/share", and got this list.

{% highlight bash %}
$ cd hotspot-87ee5ee27509/src/share
$ grep -r -l javaTimeNanos *
vm/c1/c1_LIRGenerator.cpp
vm/c1/c1_Runtime1.cpp
vm/classfile/altHashing.cpp
vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepGeneration.cpp
vm/gc_implementation/parallelScavenge/psMarkSweep.cpp
vm/gc_implementation/parallelScavenge/psParallelCompact.cpp
vm/gc_implementation/parNew/parNewGeneration.cpp
vm/memory/defNewGeneration.cpp
vm/memory/genCollectedHeap.cpp
vm/memory/genMarkSweep.cpp
vm/memory/referenceProcessor.cpp
vm/opto/library_call.cpp
vm/prims/jvm.cpp
vm/prims/jvmtiEnv.cpp
vm/runtime/os.hpp
vm/runtime/safepoint.cpp
vm/runtime/thread.cpp
{% endhighlight %}

The first few files are to support java's `System.nanoTime()` and for GC. So let's ignore them.
I'll look into share/vm/memory/XXX and share/vm/runtime/{safepoint,thread}.cpp next.
