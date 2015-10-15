#### 1. Make your Heap Explicit

**-Xms<heap size>[g|m|k] -Xmx<heap size>[g|m|k]**
To avoid dynamic heap resizing and lags, which could be caused by this, we explicitely specify minimal and maximal heap size. 
Thus Java VM will spend time only once to commit on all the heap.

**-XX:PermSize=<perm gen size>[g|m|k] -XX:MaxPermSize=<perm gen size>[g|m|k]**
Logic for permanent generation is the same as for overall heap — predefine sizing to avoid costs of dynamic changes. 
Not applicable to Java >= 8.

**-XX:MaxMetaspaceSize=<metaspace size>[g|m|k]**
By default Metaspace in Java VM 8 is not limited, though for the sake of system stability it makes sense to limit it 
with some finite value.

**-Xmn<young size>[g|m|k]**
Explicitely define size of the young generation.

**-XX:SurvivorRatio=<ratio>**
Ratio which determines size of the survivour space relatively to eden size. Ratio can be calculated using following formula:

survivor ratio=young sizesurvivor / size−2

Learn more and this one too.

#### 2. Make GC Right

**-XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled**

As response time is critical for server application concurrent collector feets best for Web applications.
Unfortunatelly G1 is still not production ready, thus we have to use Concurrent Mark-Sweep collector.

**-XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=<percent>**

By default CMS GC uses set of heuristic rules to trigger garbage collection. This makes GC less predictable and usually tends
to delay collection until old generation is almost occupied. Initiating it in advance allows to complete collection before old
generation is full and thus avoid Full GC (i.e. stop-the-world pause). 

**-XX:+UseCMSInitiatingOccupancyOnly**
Prevent usage of GC heuristics. 

**-XX:CMSInitiatingOccupancyFraction**

Informs Java VM when CMS should be triggered. Basically, it allows to create a buffer in heap, which can be filled with data,
while CMS is working. Thus percent should be back calculated from the speed in which memory is consumed in old generation
during production load. Such percent should be chosen carefully, if it will be small — CMS will work to often, if it will be
to big — CMS will be triggered too late and concurrent mode failure may occur. Usually -XX:CMSInitiatingOccupancyFraction
should be at the level 70, which mean that application should utilize less that 70% of old generation.


**-XX:+ScavengeBeforeFullGC -XX:+CMSScavengeBeforeRemark**

Instructs garbage collector to collect young generation before doing Full GC or CMS remark phase and as a result improvde
their performance due to absence of need to check references between young generation and tenured.

#### 3. Dump on Out of Memory

**-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=<path to dump>`date`.hprof**

If your application would ever fail with out-of-memory in production, you would not want to wait for another chance to
reproduce the problem. These options instruct Java VM to dump memory into file, when OOM occurred. It can cause considerable
pauses on big heaps during OOM event. However, if Java VM is at OOM already, collecting as much information about the issue is more important than trying to serve traffic with completely broken application. We are also making sure the dump would be
created with name unique per application start time (this is requeired because Java VM will fail to overwrite existing file).

#### Notes
- CMS is not always the best choice. Distributed processing applications and "batch" applications, that don't need responsiveness, but rather need to crunch several gigs of data as fast as possible, will likely do better with -XX:+ UseParallelOldGC

#### References
1. [Java VM Options You Should Always Use in Production](http://blog.sokolenko.me/2014/11/javavm-options-production.html)
2. [Human readable JVM GC timestamps](http://176.34.122.30/blog/2010/05/26/human-readable-jvm-gc-timestamps/) 
