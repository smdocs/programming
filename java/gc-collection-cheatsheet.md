GC Cheatsheet
================
>See [JVM Options Best Practices](jvm-options-notes.md) for more information on how to configure your JVM.

####- GC Stats Enabled
Basic command to start collecting GC stats
```bash
  $ -Xloggc:<file_name> –XX:+PrintGCDetails -XX:+PrintGCDateStamps
```
The JVM will start writing garbage collection messages to the file we set with the parameter -Xlogcc. 
The messages should be something like the output below which means
**[Full GC $before -> $after($total), $time in secs]**

```bash
2010-04-22T18:12:27.796+0200: 22.317: [GC 59030K->52906K(97244K), 0.0019061 secs]
2010-04-22T18:12:27.828+0200: 22.348: [GC 59114K->52749K(97244K), 0.0021595 secs]
2010-04-22T18:12:27.859+0200: 22.380: [GC 58957K->53335K(97244K), 0.0022615 secs]
2010-04-22T18:12:27.890+0200: 22.409: [GC 59543K->53385K(97244K), 0.0024157 secs]
```

Note: The bold part is simply the date and time when reported garbage collection event starts.
####- JVM Memory Model

The following figure shows the schematic representation of the underlying JVM memory model

![][jvm-mem-model-image]

>Things to remember about Young generation.
- All new allocation happens in eden. It only costs a pointer bump.
- When eden fills up, we have stop-the-world-gc into survivor space.
- Dead objects cost zero to collect.
- After several collections survivors grt tenured into old generation.

#### - Memory Sizing Options

1. Boolean options are turned on with -XX:+<option> and turned off with -XX:-<option>.
2. Numeric options are set with -XX:<option>=<number>. Numbers can include 'm' or 'M' for megabytes, 'k' or 'K' for kilobytes, and 'g' or 'G' for gigabytes (for example, 32k is the same as 32768).
3. String options are set with -XX:<option>=<string>, are usually used to specify a file, a path, or a list of commands


```bash
# 1. set initial size of the JVM heap (Yound + Old)
# Sets the initial size (in bytes) of the heap. This value must be a multiple of 1024 and greater than 1 MB. 
# Append the letter # k or K to indicate kilobytes, m or M to indicate megabytes, g or G to indicate 
# gigabytes. 
-Xms256m Or --XX:InitialHeapSize=256m

# set the MAX size of JVM Heap (Young + Old)
-Xmx2g OR -XX:MaxHeapSize=2g
```
> Note: If you do not set this option, then the initial size will be set as the sum of the sizes allocated for the old
generation and the young generation. The initial size of the heap for the young generation can be set using the -Xmn 
option or the -XX:NewSize option.

- Difference between NewSize and MaxNewSize options

The young generation is set by a policy that bounds the size from below by NewSize and bounds it from above by MaxNewSize. As
the young generation grows from NewSize to MaxNewSize, both eden and the survivor spaces grow.

```bash
## Set the initial new size of NEW size
-XX:NewSize=64m
## Set the maximum new size or upperlimit of New
-XX:MaxNewSize=64m
```
>Note: If all the live objects in eden do no fit into a survivor space, the remaining live objects are promoted into the old generation.

```bash
 ## alternative way to specify size of the young space.
 -XX:NewRatio=3 //This means that the Young space will be 2 times SMALLER than the Old space, 
 i.e. 1/3 of heap size.
```
 Ratio which determines size of the survivour space relatively to eden size. Ratio can be calculated using following 
 formula:
 
 ![][survivor-eq-image]
 
 JVM option to set survivor ratio
 ```bash
 ## Set the size of a single survivor space relative to eden space size.
 -XX:SurvivorRatio=15 //
 ```
 >e.g. -XX:NewSize=64 -XXSurvivorRatio=6 means that each survivor space will be 8m (i.e. 8 * 2 survivor space = 16m) and the Eden space will be 48m
 
 #### Performance Tuning checklist
 
 Tuning for throughput and Latency. As you would appreciate by now, tuning Java servers is a tricky business, and there is a lot of ice hidden below the waterline in that iceberg. Following is a summary checklist of the steps we discussed. This may not tell you how to profile everything, but it can save you from many pitfalls.
 
1. Check the load average of the machine. If it is > 4XNumberOfCores, then your machine is running at full throttle and you can skip to the CPU tuning part.

2. Are you hitting the system hard enough? Try simulate more client by increasing the number of threads in your client programs. If that improves throughput then increase the load until throughput is maximized.

3. Are your threads idling? If you have too many locks or too few threads, the system might not provide full throughput. Use a profiler to view the lock profile and try to remove them. While some locks are neededmost are not. 
4. Try increasing the number of  threads in your thread pools, and check  whether  that improves throughput.
    - Now check the CPU profile for hotspots
    - Study the tree view and make sure any hot spots are where expected. For example, an  XML parser would be expected to consume a  lot of CPU. But if you see something unexpected taking too much CPU, there is something to fix

5. Check memory and GC, if GC throughput is less than 90% it is time to tune the GC. If memory runs on the brink (check for paging), add more and see if that helps.

6. Look at DB profiles, and ensure that what takes most load are expected.
7. Look at network IO profile. Go to hot spots, make sure all major writes are what you expect. If there is some unnecessary IO, this will show it.
8. Verify that resources such as underlying network and disk mounting are in place
 

####References
- [JVM Performance Tuning](http://www.slideshare.net/aszegedi/everything-i-ever-learned-about-jvm-performance-tuning-twitter)
- [Tuning Java Servers](http://www.infoq.com/articles/Tuning-Java-Servers)
- [Understanding Linux Load average](https://prutser.wordpress.com/2012/04/23/understanding-linux-load-average-part-1/)
[jvm-mem-model-image]: resources/jvm-mem-model.png
[survivor-eq-image]:resources/survivor-ratio.png
