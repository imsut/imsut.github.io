---
layout: post
title: get cache specs via sysfs
---

By reading some (virtual) files in sysfs, you can get to know cache architecture on your PC in detail.

My PC has Core i5 M540, which has 2 core, with HyperThreading enabled. Therefore Linux kernel shows us 4 CPUs, and there are four directories, cpu[0-3], in /sys/devices/system/cpu. The four logical cores are symmetric, however, I'll list files in /sys/devices/system/cpu/cpu0/cache only.

The directory /sys/devices/system/cpu/cpu0/cache has four entries: index[0-3], each of which seems to represent L1 D-cache, L1 I-cache, L2 cache, L3 cache.

{% highlight bash %}
$ head index[0-3]/{level,type}
==> index0/level <==
1

==> index1/level <==
1

==> index2/level <==
2

==> index3/level <==
3

==> index0/type <==
Data

==> index1/type <==
Instruction

==> index2/type <==
Unified

==> index3/type <==
Unified
{% endhighlight %}


L1 cache is devided into D-cache and I-cache, while L2 and L3 are unified one. Two files, size and ways_of_associativity, tell you the size and associativity as follows. 

{% highlight bash %}
% head index[0-3]/{size,ways_of_associativity}       
==> index0/size <==
32K

==> index1/size <==
32K

==> index2/size <==
256K

==> index3/size <==
3072K

==> index0/ways_of_associativity <==
8

==> index1/ways_of_associativity <==
4

==> index2/ways_of_associativity <==
8

==> index3/ways_of_associativity <==
12
{% endhighlight %}

L1 D-cache and I-cache are same in size, but D-cache has higher associativity, that is, D-cache contains up to 8 data which have the same index (lower bit of address) while I-cache can contain at most 4 data with common index. shared_cpu_list indicates which logical core shares the cache. 

{% highlight bash %}
% head index[0-3]/shared_cpu_list   
==> index0/shared_cpu_list <==
0-1

==> index1/shared_cpu_list <==
0-1

==> index2/shared_cpu_list <==
0-1

==> index3/shared_cpu_list <==
0-3
{% endhighlight %}

In this case, L1 and L2 cache are used by cpu0 and cpu1 which are two threads in a single core. L3 cache are shared by all four threads, or 2 cores.


As you can see, Linux kernel shows you detailed data about CPU cache. It would be better and easier for you to search sysfs first before searching Intel website.
